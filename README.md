# AWS VPC Terraform Setup

This Terraform configuration deploys a production-ready VPC with the following features:

## Features

- **VPC Setup**: Create a VPC with a CIDR block of `10.0.0.0/16`.
- **Public Subnet**: A public subnet with an associated Internet Gateway (IGW) to allow internet access.
- **Private Subnets**: Two private subnets, each in separate Availability Zones (AZ), with a NAT Gateway deployed in the public subnet to allow private subnets to access the internet securely.
- **High Availability**: Resources are distributed across multiple Availability Zones to ensure high uptime for production workloads.
- **EC2 Instance on the Public Subnet**:  
  - Deployed on the public subnet.  
  - Has an **SSH security group** allowing access.  
  - Uses a **new key pair** for secure SSH access.  
  - Runs **custom userdata scripts** for initialization on launch.

---

## Architecture Overview

                                 ┌───────────────┐
                                 │   Internet    │
                                 └──────┬────────┘
                                        │
                                  (via IGW)
                                        │
                               ┌────────▼────────┐
                               │ Internet Gateway│
                               └────────┬────────┘
                                        │
                                        │
                               ┌────────▼────────┐
                               │    Public Subnet │
                               │    10.0.1.0/24   │
                               │ Availability AZa │
                               │ map_public_ip_on_launch
                               └────────┬────────┘
                                        │
                   ┌────────────────────┼────────────────────┐
                   │                    │                    │
                   │                    │                    │
        ┌──────────▼────────┐ ┌─────────▼────────┐ ┌────────▼────────┐
        │ NAT Gateway + EIP │ │ Private Subnet 1 │ │ Private Subnet 2 │
        │ (in Public Subnet)│ │   10.0.2.0/24    │ │   10.0.3.0/24    │
        │ Availability AZa  │ │ Availability AZb │ │ Availability AZc │
        └──────────┬────────┘ └─────────┬────────┘ └────────┬────────┘
                   │                    │                    │
                   │                    │                    │
             Outbound            Outbound only         Outbound only
            Internet                Internet              Internet
             Access                  Access               Access
         (via NAT GW)            (via NAT GW)         (via NAT GW)


- **Public Subnet (10.0.1.0/24 @ AZ a)**  
  - Hosts the **Internet Gateway** and **NAT Gateway**  
  - Automatically assigns public IPs  
  - Allows direct inbound & outbound internet traffic  

- **Private Subnets (10.0.2.0/24 @ AZ b, 10.0.3.0/24 @ AZ c)**  
  - No public IPs  
  - Route all outbound traffic through the **NAT Gateway** in the public subnet  
  - Fully isolated from direct inbound internet access  

This design ensures:
- **High availability** across three AZs  
- **Public resources** (NAT, bastion, web services) reach the internet directly  
- **Private resources** (databases, internal apps) can only initiate outbound connections  

