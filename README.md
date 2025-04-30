# AWS VPC Terraform Setup

This Terraform configuration deploys a production-ready VPC with the following features:

- **Public subnet** with an Internet Gateway  
- **Two private subnets**, each routed through a NAT Gateway in the public subnet  
- Subnets spread across **multiple Availability Zones** (`il-central-1a`, `il-central-1b`, `il-central-1c`) for high availability  
- Tags and naming conventions to keep resources organized  

---

## Architecture Overview

┌─────────────────────────────────────────────────────────┐ │ VPC 10.0.0.0/16 │ │ │ │ ┌───────────────────────┐ ┌───────────────────────┐ │ │ │ Public Subnet │ │ Private Subnet 1 │ │ │ │ 10.0.1.0/24 (AZ a) │◄─► IGW│ 10.0.2.0/24 (AZ b) │ │ │ │ NAT Gateway + EIP │ │ NAT via public subnet │ │ │ └───────────────────────┘ └───────────────────────┘ │ │ ▲ │ │ │ │ │ ┌───────────────────────┐ │ │ │ │ Private Subnet 2 │────────────┘ │ │ │ 10.0.3.0/24 (AZ c) │ NAT via public subnet │ │ └───────────────────────┘ │ └─────────────────────────────────────────────────────────┘
