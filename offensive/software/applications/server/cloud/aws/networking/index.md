---
title: "AWS networking"
description: "Abusing AWS networking to reach and expose resources: over-permissive security groups, internet-exposed services, VPC reachability and peering, and the DNS and endpoint misconfigurations that widen access inside an account."
keywords:
  - security group
  - VPC
  - exposed services
  - peering
  - networking
---

# Networking

AWS networking decides what is reachable: **security groups** and NACLs that are too open, services bound to the internet, and **VPC** peering and endpoints that let a foothold in one network reach another. Networking rarely compromises an account on its own, but it is how a reachable foothold turns into reach across the environment.

The pages here cover finding **internet-exposed** services and over-permissive **security groups**, mapping **VPC** reachability and peering to pivot between networks, and the endpoint and DNS misconfigurations that expose internal services.

## What folds in here

- **Lateral movement** at the network layer (reaching another VPC or an internal service); the identity-based cross-account movement lives in [identity](../identity/index.md).

## References

- [HackTricks Cloud: AWS VPC and networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: security groups for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
