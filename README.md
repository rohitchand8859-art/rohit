# 🌐 AWS VPC Peering (Same Account, Different Regions)

This project demonstrates how to **create and configure a VPC Peering Connection** between two **VPCs in the same AWS account but located in different regions**. It includes step-by-step instructions, architecture explanation, and configuration details using the AWS Management Console and AWS CLI.

---

## 🧠 What is VPC Peering?

**VPC Peering** is a **networking connection between two Virtual Private Clouds (VPCs)** that allows you to **route traffic between them privately** using private IP addresses — as if they were part of the same network.

It is commonly used for:
- Sharing resources between VPCs (like EC2, RDS, etc.)
- Multi-region communication within the same organization
- Secure inter-VPC communication without using the public internet

---

## 🏗️ Architecture Overview

