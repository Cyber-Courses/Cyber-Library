---
title: "Direct Connect: bridging on-premises networks into the VPC"
description: "Abusing Direct Connect links and virtual interfaces that bridge on-premises networks into the VPC."
keywords:
  - Direct Connect
  - virtual interface
  - hybrid
  - on-premises
  - VPC
---

# Direct Connect

Direct Connect is a dedicated link between an on-premises network and AWS, exposed through virtual interfaces (VIFs) that attach to VPCs or to a Direct Connect gateway. A foothold on either side of a VIF reaches the other: a compromised VPC role reads the hybrid routing that leads on-premises, and a compromised on-premises host rides the private VIF into VPC ranges that assume the link is trusted.

## Enumerating links and interfaces

```bash
aws directconnect describe-connections
aws directconnect describe-virtual-interfaces \
  --query "virtualInterfaces[].[virtualInterfaceId,virtualInterfaceType,vlan,amazonAddress,customerAddress,virtualGatewayId]"
aws directconnect describe-direct-connect-gateways
```

## Exploitation notes

- Private VIF address pairs (`amazonAddress` / `customerAddress`) reveal the hybrid link's inside ranges to scan from a VPC foothold.
- Traffic over a private VIF is often implicitly trusted on both sides, so controls that assume an internet boundary do not apply.
- A Direct Connect gateway can attach the link to multiple VPCs and accounts, widening the blast radius of either side's compromise.

## Tools

- **AWS CLI** (`directconnect describe-*`): link, VIF, and gateway enumeration.
- **Route/VPC analysis** ([VPC](vpc.md)): correlate VIF-advertised ranges with reachable subnets.

## References

- [AWS: Direct Connect virtual interfaces](https://docs.aws.amazon.com/directconnect/latest/UserGuide/WorkingWithVirtualInterfaces.html)
- [HackTricks Cloud: AWS hybrid networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
