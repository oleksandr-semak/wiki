---
layout: wiki
title: Transit Gateway for Centralized NAT Gateway
permalink: /wiki/aws/topics/network/transit-gateway-central-nat/
tags: [aws, vpc, network, transit-gateway, nat-gateway, multi-account]
---

## The Problem

In multi-VPC architectures, each VPC typically needs its own NAT Gateway for outbound internet access:
- NAT Gateway per AZ per VPC multiplies costs
- Each VPC requires an Internet Gateway
- Duplicate infrastructure across accounts/VPCs
- No centralized control over egress traffic

> **Warning:** NAT Gateway pricing includes both hourly charges ($0.045/hr) and data processing fees ($0.045/GB). With multiple VPCs, costs grow linearly — a 10-VPC architecture could cost $324/month in NAT Gateway hourly fees alone, before any data transfer.
{: .warning}

## The Solution: Centralized NAT with Transit Gateway

**AWS Transit Gateway** enables a hub-and-spoke architecture where a single "egress VPC" handles all outbound internet traffic. Spoke VPCs route through Transit Gateway to the central NAT Gateway — eliminating redundant NAT infrastructure.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Egress VPC (Hub)                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │ NAT Gateway │    │ NAT Gateway │    │     IGW     │      │
│  │   (AZ-a)    │    │   (AZ-b)    │    │             │      │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘      │
│         │                  │                   │             │
│  ┌──────┴──────────────────┴───────────────────┴──────┐     │
│  │              Public Subnets                         │     │
│  └─────────────────────┬───────────────────────────────┘     │
│  ┌─────────────────────┴───────────────────────────────┐     │
│  │           TGW Attachment Subnets                    │     │
│  └─────────────────────┬───────────────────────────────┘     │
└────────────────────────┼────────────────────────────────────┘
                         │
              ┌──────────┴──────────┐
              │   Transit Gateway   │
              └──────────┬──────────┘
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Spoke VPC1 │  │  Spoke VPC2 │  │  Spoke VPC3 │
│  (Private)  │  │  (Private)  │  │  (Private)  │
└─────────────┘  └─────────────┘  └─────────────┘
```

## Key Components

| Component | Purpose |
|-----------|---------|
| Transit Gateway | Central hub connecting all VPCs |
| Egress VPC | Contains NAT Gateway(s) and IGW |
| TGW Attachment Subnet | Dedicated subnet for TGW ENIs |
| Spoke VPCs | Private VPCs without NAT/IGW |

## Routing Configuration

### Spoke VPC Route Tables
```
Destination      Target
0.0.0.0/0        tgw-xxxxx (Transit Gateway)
10.0.0.0/8       tgw-xxxxx (for inter-VPC traffic)
```

### Egress VPC - TGW Attachment Subnet
```
Destination      Target
0.0.0.0/0        nat-xxxxx (NAT Gateway)
10.0.0.0/8       tgw-xxxxx (return traffic to spokes)
```

### Egress VPC - Public Subnet
```
Destination      Target
0.0.0.0/0        igw-xxxxx (Internet Gateway)
10.0.0.0/8       tgw-xxxxx
```

### Transit Gateway Route Table
```
Destination      Attachment
0.0.0.0/0        Egress VPC attachment
10.1.0.0/16      Spoke VPC1 attachment
10.2.0.0/16      Spoke VPC2 attachment
```

## Key Benefits

- **Cost Reduction** — single NAT Gateway set instead of one per VPC
- **Centralized Egress** — single point for security inspection/logging
- **Simplified Architecture** — spoke VPCs need no public subnets
- **Scalable** — add new VPCs without additional NAT infrastructure
- **Security** — centralized firewall/IDS integration point

## Important Considerations

> **Note:** TGW Attachment Subnets must be separate from public subnets. This prevents asymmetric routing where return traffic bypasses the NAT Gateway.
{: .info}

- **Data Transfer Costs** — TGW charges $0.02/GB processed
- **Cross-AZ Traffic** — place TGW attachments in each AZ used
- **Appliance Mode** — enable for stateful inspection appliances
- **Bandwidth** — NAT Gateway supports up to 100 Gbps

## Use Cases

1. **Multi-Account Landing Zone** — centralized egress for all workload accounts
2. **Cost Optimization** — reduce NAT Gateway proliferation
3. **Security Compliance** — inspect/log all egress traffic centrally
4. **Hybrid Networks** — combine with Direct Connect/VPN at hub

---

> **Read the full guide:** [Creating a single internet exit point from multiple VPCs Using AWS Transit Gateway](https://aws.amazon.com/blogs/networking-and-content-delivery/creating-a-single-internet-exit-point-from-multiple-vpcs-using-aws-transit-gateway/) — AWS Networking Blog
{: .article-link}
