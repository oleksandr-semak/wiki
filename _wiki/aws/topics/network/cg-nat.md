---
layout: wiki
title: EKS IPv4 Address Exhaustion & CG-NAT
permalink: /wiki/aws/topics/network/cg-nat/
tags: [aws, eks, vpc, network]
---

## The Problem

Large-scale Amazon EKS clusters can quickly exhaust available IPv4 addresses. Each pod requires an IP from your VPC CIDR, and with hundreds or thousands of pods, you may run out of RFC 1918 private addresses — especially when your VPC CIDR is constrained by organizational IP allocation policies.

> **Warning:** This issue is particularly common when **CNI Prefix Delegation** is enabled. While prefix delegation improves IP allocation efficiency by assigning /28 prefixes instead of individual IPs, it can accelerate IP exhaustion if your subnet CIDR is not sized appropriately. Be careful with this setting in production environments.
{: .warning}

## The Solution: Private NAT Gateway with CG-NAT

**CG-NAT (Carrier-Grade NAT)** uses the `100.64.0.0/10` address range (defined in RFC 6598) — a special block reserved for shared address space. By combining this with AWS Private NAT Gateway, you can:

- Extend your IP address pool without conflicting with existing RFC 1918 ranges
- Enable communication between overlapping CIDR blocks
- Scale EKS clusters without exhausting corporate IP allocations

## Key Concepts

| Range | Purpose |
|-------|---------|
| `10.0.0.0/8` | RFC 1918 - Private |
| `172.16.0.0/12` | RFC 1918 - Private |
| `192.168.0.0/16` | RFC 1918 - Private |
| `100.64.0.0/10` | RFC 6598 - CG-NAT Shared |

## When to Use

- EKS clusters running out of IP addresses
- Multi-VPC architectures with overlapping CIDRs
- Organizations with limited IP allocation from central IT

---

> **Read the full guide:** [Addressing IPv4 address exhaustion in Amazon EKS clusters using private NAT gateways](https://aws.amazon.com/blogs/containers/addressing-ipv4-address-exhaustion-in-amazon-eks-clusters-using-private-nat-gateways/) — AWS Containers Blog
{: .article-link}
