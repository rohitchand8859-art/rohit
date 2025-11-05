# 🌐 AWS VPC Peering (Same Account, Different Regions)  
> Full setup with public/private subnets and EC2 instances in each VPC

This project demonstrates how to **create two VPCs** (each with one public and one private subnet), **launch EC2 instances**, and **configure VPC Peering** between them — all within the **same AWS account** but across **different regions**.  

---

## 🧠 What is VPC Peering?

**VPC Peering** connects two Virtual Private Clouds so that they can communicate using **private IP addresses**.  
It allows secure traffic flow without using the public internet.

Use cases:
- Multi-region application communication  
- Resource sharing across VPCs  
- Isolating environments but allowing internal communication  

---

## ⚙️ Step-by-Step Setup Guide

### **Step 1: Create VPCs**

#### VPC-A (Region 1: ap-south-1)
1. Go to **VPC Console → Your VPCs → Create VPC**
2. Choose **VPC only**.
3. Name: `VPC-A`
4. IPv4 CIDR block: `10.0.0.0/16`
5. Leave IPv6 and tenancy default → **Create VPC**

#### VPC-B (Region 2: us-east-1)
1. Switch to **Region 2**
2. Name: `VPC-B`
3. IPv4 CIDR block: `172.16.0.0/16`
4. Create

---

### **Step 2: Create Subnets**

#### VPC-A Subnets
| Name | CIDR | AZ | Purpose |
|------|------|----|----------|
| `Public-A` | `10.0.1.0/24` | ap-south-1a | Public subnet |
| `Private-A` | `10.0.2.0/24` | ap-south-1b | Private subnet |

#### VPC-B Subnets
| Name | CIDR | AZ | Purpose |
|------|------|----|----------|
| `Public-B` | `172.16.1.0/24` | us-east-1a | Public subnet |
| `Private-B` | `172.16.2.0/24` | us-east-1b | Private subnet |

---

### **Step 3: Create Internet Gateways and Attach**

For each VPC:
1. Go to **Internet Gateways → Create Internet Gateway**
   - Name: `IGW-A` and `IGW-B`
2. Attach to their respective VPCs (`VPC-A` and `VPC-B`)

---

### **Step 4: Create Route Tables**

#### For VPC-A:
- **Public Route Table:**  
  - Associate with **Public-A subnet**  
  - Add route: `0.0.0.0/0 → IGW-A`

- **Private Route Table:**  
  - Associate with **Private-A subnet**  
  - (Later we’ll add peering route here)

#### For VPC-B:
- **Public Route Table:**  
  - Associate with **Public-B subnet**  
  - Add route: `0.0.0.0/0 → IGW-B`

- **Private Route Table:**  
  - Associate with **Private-B subnet**

---

### **Step 5: Launch EC2 Instances**

In each region:
1. Go to **EC2 → Launch Instance**
2. Choose Amazon Linux 2 AMI (free tier)
3. Choose **t2.micro**
4. Configure subnet and network:
   - For public subnet → assign public IP
   - For private subnet → no public IP
5. Security Group:
   - Allow SSH (22) from your IP
   - Allow ICMP (for ping testing)
6. Launch instance:
   - `EC2-PubA`, `EC2-PrivA` in VPC-A
   - `EC2-PubB`, `EC2-PrivB` in VPC-B

---

### **Step 6: Create VPC Peering**

From **Region 1 (ap-south-1)**:
1. Go to **VPC → Peering Connections → Create Peering Connection**
2. Name: `VPC-A-VPC-B`
3. Requester VPC: `VPC-A`
4. Peer VPC: **Another VPC in my account**
5. Peer Region: `us-east-1`
6. Peer VPC ID: `VPC-B`
7. Create

Then in **Region 2 (us-east-1)**:
- Go to Peering Connections → Select request → **Accept Request**

---

### **Step 7: Update Route Tables for Peering**

| VPC | Route Table | Destination | Target |
|-----|--------------|--------------|--------|
| VPC-A | Private Route Table | `172.16.0.0/16` | Peering Connection ID |
| VPC-B | Private Route Table | `10.0.0.0/16` | Peering Connection ID |

> 💡 Add routes to both private and public route tables if needed for both subnet types.

---

### **Step 8: Modify Security Groups**

On **all instances**, allow inbound traffic from the peer VPC CIDR.

Example rules:
| Type | Protocol | Port | Source |
|------|-----------|------|--------|
| SSH | TCP | 22 | Peer VPC CIDR |
| ICMP | All | All | Peer VPC CIDR |

---

## 🔗 Step 10: Test Connectivity Between VPCs

Now that the peering connection and route tables are configured, test communication between EC2 instances across VPCs.

### 🖥️ 1️⃣ Connect to EC2 Instance in VPC-A
If the instance is in a **public subnet**, connect directly using SSH:
bash
ssh -i mykey.pem ec2-user@<EC2-Public-IP>
If it’s in a private subnet, first connect to a bastion host in the same VPC.

🧪 2️⃣ Ping Private Instance in VPC-B
From inside EC2-PrivA (VPC-A):

bash
Copy code
ping 172.16.2.10
✅ If the configuration is correct, you’ll receive ICMP replies like:

nginx
Copy code
PING 172.16.2.10 (172.16.2.10) 56(84) bytes of data.
64 bytes from 172.16.2.10: icmp_seq=1 ttl=254 time=52 ms
If the ping fails:

Check if ICMP is allowed in both instances’ Security Groups.

Verify route tables include the correct Peering Connection.

Confirm that the VPC Peering status is Active.

🔗 Step 11: Connect EC2 Instance from One VPC to Another (Private IP Communication)
After verifying connectivity, you can now directly SSH between EC2 instances using private IPs over VPC Peering.

🧾 Example Setup
Component	VPC-A (ap-south-1)	VPC-B (us-east-1)
VPC CIDR	10.0.0.0/16	172.16.0.0/16
Private Subnet	10.0.2.0/24	172.16.2.0/24
EC2 Instance	EC2-PrivA (10.0.2.10)	EC2-PrivB (172.16.2.10)

🖥️ Connect from EC2-PrivA → EC2-PrivB
Once you’re inside the private EC2 in VPC-A, connect to the EC2 in VPC-B using its private IP:

bash
Copy code
ssh ec2-user@172.16.2.10
✅ You are now connected from VPC-A → VPC-B over AWS’s private backbone network (no Internet access used).

🧪 Test with Ping
You can also verify the connection with ping:

bash
Copy code
ping 172.16.2.10
If you get replies, communication is working successfully over VPC Peering.

---

## ⚙️ Requirements

Before starting this setup, ensure you have the following:

### 🧰 AWS Requirements
- An **AWS account** with sufficient permissions to create:
  - VPCs  
  - Subnets  
  - Internet Gateways  
  - Route Tables  
  - EC2 Instances  
  - VPC Peering Connections
- At least **two different AWS regions** available (example: `ap-south-1` and `us-east-1`)
- Basic understanding of:
  - Networking (CIDR, routing)
  - Security Groups and NACLs
  - SSH key management

## 👨‍💻 **Author**

Rohit Chand


Cloud Computing Enthusiast|AWS Learner


