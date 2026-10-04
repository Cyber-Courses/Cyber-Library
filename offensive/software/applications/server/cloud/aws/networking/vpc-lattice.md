---
title: "VPC Lattice: reaching services across VPCs through auth policies"
description: "Abusing VPC Lattice service networks and auth policies to reach services across VPCs and accounts."
keywords:
  - VPC Lattice
  - service network
  - auth policy
  - cross-account
  - service mesh
---

# VPC Lattice

VPC Lattice connects services across VPCs and accounts through a service network, with access governed by IAM-style auth policies rather than by network routing. A permissive auth policy (a wildcard principal, or no policy at all) lets any principal that can reach the service network call the registered services, turning Lattice into a cross-account application-layer pivot.

## Enumerating the mesh

```bash
aws vpc-lattice list-service-networks
aws vpc-lattice list-services
aws vpc-lattice list-service-network-service-associations --service-network-identifier <id>
aws vpc-lattice get-auth-policy --resource-identifier <service-or-network-arn>
```

## Exploitation notes

- An auth policy with `"Principal":"*"` or `AWS:"*"` and no condition exposes the service to any authenticated caller on the network.
- Lattice bypasses traditional security-group and subnet boundaries, so a service unreachable by routing may still be callable through the service network.
- Service associations across accounts make a permissive policy a cross-account reach, not just an in-account one.

## Tools

- **AWS CLI** (`vpc-lattice ...`): service-network, service, and auth-policy enumeration.
- **ScoutSuite** / **Prowler**: flag permissive Lattice auth policies.

## References

- [AWS: VPC Lattice auth policies](https://docs.aws.amazon.com/vpc-lattice/latest/ug/auth-policies.html)
- [HackTricks Cloud: AWS networking services](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [CloudFox: mapping reachable cloud network paths](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
