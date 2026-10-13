---
title: "Azure networking"
order: 7
description: "Attacking Azure networking: network security groups and VNet exposure, private endpoints, DNS takeover, Front Door and CDN, and Bastion."
keywords:
  - Azure networking
  - NSG
  - VNet
  - DNS takeover
  - private endpoint
---

# Networking

Azure network reachability decides what a foothold can touch. Network security groups gate traffic, VNets and peerings define the internal blast radius, private endpoints pull PaaS services onto internal addresses, and the edge services (Front Door, CDN, Bastion) and DNS records are each abusable in their own right. Enumeration here turns the subscription's network posture into a target list and a pivot map.

## What folds in here

- **[Network security groups](network-security-groups.md)**: reading and rewriting NSG rules to expose or reach filtered resources.
- **[VNet](vnet.md)**: peering, service endpoints, and private access to pivot between subnets.
- **[Private endpoints](private-endpoints.md)**: Private Link to reach PaaS resources over internal addresses.
- **[DNS takeover](dns-takeover.md)**: claiming dangling records from deleted App Service, Traffic Manager, CDN, or public-IP resources.
- **[Front Door and CDN](front-door-and-cdn.md)**: origin exposure, host-header and routing abuse, and dangling endpoints.
- **[Bastion](bastion.md)**: pivoting into private VMs over RDP and SSH.

## References

- [HackTricks Cloud: Azure networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Azure networking documentation](https://learn.microsoft.com/azure/networking/)
