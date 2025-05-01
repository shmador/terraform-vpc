# AWS VPC Terraform Setup

This Terraform configuration deploys a production-ready VPC with the following features:

## Features

- **VPC**: `10.0.0.0/16` CIDR block.  
- **Public Subnet** (AZ a):  
  - Internet Gateway (IGW)  
  - EC2 instance with SSH SG, dedicated key pair, and custom userdata  
- **Private Subnets** (AZ b & c):  
  - No public IPs  
  - Outbound internet via NAT Gateway in public subnet  
- **High Availability**: Resources spread across three AZs for resilience  


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

