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



