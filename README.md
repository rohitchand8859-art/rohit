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
Region 1 (ap-south-1) Region 2 (us-east-1)
┌────────────────────────────┐ ┌────────────────────────────┐
│ VPC-A (10.0.0.0/16) │ │ VPC-B (172.16.0.0/16) │
│ │ │ │
│ ┌────────────┐ │ │ ┌────────────┐ │
│ │PublicSubnet│10.0.1.0/24 │ │ │PublicSubnet│172.16.1.0/24│
│ │ EC2-PubA │ │ │ │ EC2-PubB │ │
│ └────────────┘ │ │ └────────────┘ │
│ ┌────────────┐ │ │ ┌────────────┐ │
│ │PrivateSubnet│10.0.2.0/24│ │ │PrivateSubnet│172.16.2.0/24│
│ │ EC2-PrivA │ │ <── Peering ─> │ EC2-PrivB │ │
│ └────────────┘ │ │ └────────────┘ │
└────────────────────────────┘ └────────────────────────────┘


