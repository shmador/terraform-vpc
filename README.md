# AWS VPC Terraform Setup

This Terraform configuration deploys a production-ready VPC with the following features:

- **Public subnet** with an Internet Gateway  
- **Two private subnets**, each routed through a NAT Gateway in the public subnet  
- Subnets spread across **multiple Availability Zones** (`il-central-1a`, `il-central-1b`, `il-central-1c`) for high availability  
- Tags and naming conventions to keep resources organized  

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

