---
title: "AWS networking"
order: 7
description: "Abusing AWS networking to reach and expose resources: over-permissive security groups, internet-exposed services, VPC reachability and peering, Route53 and CloudFront, dangling-DNS takeover, and the edge and hybrid services that widen access."
keywords:
  - security group
  - VPC
  - Route53
  - dangling DNS
  - networking
---

# Networking

AWS networking decides what is reachable: **security groups** that are too open, services bound to the internet, **VPC** peering and endpoints that let a foothold in one network reach another, and the DNS and edge services that front and route traffic. Networking rarely compromises an account on its own, but it is how a reachable foothold turns into reach across the environment, and how a trusted name is captured.

## What folds in here

- **[Security groups](security-groups.md)**: internet-exposed management ports and services from over-permissive ingress.
- **[VPC](vpc.md)**: peering, endpoints, and routing to pivot between networks and accounts.
- **[Route53](route53.md)**: hijacking records and poisoning resolution where you hold zone write access.
- **[CloudFront](cloudfront.md)**: origin disclosure and cache-behavior abuse to reach protected origins.
- **[Dangling-DNS takeover](dangling-dns-takeover.md)**: claiming the orphaned S3, CloudFront, or ELB resource behind a stale record.
- **[Global Accelerator](global-accelerator.md)**: listeners and endpoint groups to front endpoints behind trusted anycast IPs.
- **[VPC Lattice](vpc-lattice.md)**: permissive service-network auth policies for cross-VPC and cross-account reach.
- **[Direct Connect](direct-connect.md)**: virtual interfaces that bridge on-premises networks into the VPC.
- **[Network Firewall](network-firewall.md)**: rule-group gaps and inspection teardown.
- **[WAF](waf.md)**: web ACL bypass through encoding, size limits, and origin-direct requests.

Network-layer lateral movement lives here; the identity-based cross-account movement through `AssumeRole` lives in [identity](../identity/index.md).

## References

- [HackTricks Cloud: AWS VPC and networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [AWS: security groups for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [can-i-take-over-xyz (subdomain takeover fingerprints)](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
