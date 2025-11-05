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

### **Step 9: Enable DNS Resolution (Optional)**

To resolve private hostnames across VPCs:
```bash
aws ec2 modify-vpc-peering-connection-options \
  --vpc-peering-connection-id pcx-XXXXXX \
  --requester-peering-connection-options '{"AllowDnsResolutionFromRemoteVpc":true}' \
  --accepter-peering-connection-options '{"AllowDnsResolutionFromRemoteVpc":true}'
