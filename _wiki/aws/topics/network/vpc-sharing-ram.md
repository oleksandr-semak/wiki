---
layout: wiki
title: VPC Sharing with AWS RAM
permalink: /wiki/aws/topics/network/vpc-sharing-ram/
tags: [aws, vpc, network, multi-account]
---

## The Problem

Multi-account AWS architectures often require complex networking:
- VPC Peering between accounts
- Transit Gateway connections
- PrivateLink endpoints
- Duplicate subnets across accounts

This leads to increased costs, operational overhead, and connectivity complexity.

> **Warning:** Without VPC sharing, each account needs its own VPC with NAT Gateways, VPN connections, and Direct Connect — multiplying infrastructure costs significantly.
{: .warning}

## The Solution: VPC Sharing with AWS RAM

**AWS Resource Access Manager (RAM)** allows you to share subnets across accounts within the same AWS Organization. Participant accounts can launch resources directly into shared subnets — eliminating the need for inter-VPC connectivity.

## How It Works

| Component | Description |
|-----------|-------------|
| Owner Account | Creates VPC, subnets, and shares via RAM |
| Participant Account | Launches resources into shared subnets |
| AWS RAM | Manages resource sharing across accounts |

```
┌─────────────────────────────────────────┐
│           Owner Account (Network)        │
│  ┌─────────────────────────────────────┐ │
│  │              VPC                     │ │
│  │  ┌──────────┐  ┌──────────┐         │ │
│  │  │ Subnet A │  │ Subnet B │ (shared)│ │
│  │  └──────────┘  └──────────┘         │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
              ▼ RAM Share ▼
┌──────────────────┐  ┌──────────────────┐
│ Participant Acc1 │  │ Participant Acc2 │
│   EC2, RDS, etc  │  │   Lambda, ECS    │
└──────────────────┘  └──────────────────┘
```

## Key Benefits

- **No VPC Peering** — resources in same subnet, no routing needed
- **Reduced Costs** — single NAT Gateway, VPN, Direct Connect
- **Centralized Network** — owner controls VPC, routes, NACLs
- **Account Isolation** — each account manages own resources
- **Simplified Security** — Security Groups work across accounts

## What Can Be Shared

| Shareable | Not Shareable |
|-----------|---------------|
| Subnets | VPCs |
| Transit Gateway | Internet Gateway |
| Route 53 Resolver | NAT Gateway |
| License Manager | Security Groups |

## Use Cases

1. **Centralized Network Account** — single VPC shared across workload accounts
2. **Shared Services** — database subnets shared with app accounts
3. **Cost Optimization** — reduce NAT Gateway and Direct Connect costs
4. **Compliance** — centralized network control with distributed workloads

---

> **Read the full guide:** [VPC sharing: A new approach to multiple accounts and VPC management](https://aws.amazon.com/blogs/networking-and-content-delivery/vpc-sharing-a-new-approach-to-multiple-accounts-and-vpc-management/) — AWS Networking Blog
{: .article-link}
