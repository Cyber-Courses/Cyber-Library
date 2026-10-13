---
title: "VNet: peering and service endpoints as a pivot"
order: 2
description: "Abusing Azure virtual networks: peering, service endpoints, and private access to pivot between subnets and resources."
keywords:
  - VNet
  - peering
  - service endpoint
  - subnet
  - pivot
---

# VNet

A virtual network is the internal blast radius. From a compromised VM or container, VNet **peering** often reaches far beyond the local subnet because peerings are set up for convenience and rarely segmented, and **service endpoints** extend a subnet's identity to PaaS resources (storage, SQL) that then trust traffic from the subnet. Mapping the peering graph tells you how far one foothold reaches.

## Mapping the reachable network

```bash
# the VNets, their peerings, and subnets
az network vnet list --query "[].{Name:name,RG:resourceGroup,Space:addressSpace.addressPrefixes}" -o table
az network vnet peering list --vnet-name <vnet> -g <rg> \
  --query "[].{Name:name,Remote:remoteVirtualNetwork.id,State:peeringState}" -o table
# subnets and the service endpoints they carry
az network vnet subnet list --vnet-name <vnet> -g <rg> \
  --query "[].{Name:name,Endpoints:serviceEndpoints[].service}" -o table
```

## Exploitation notes

- A `Connected` peering with `allowForwardedTraffic`/`allowGatewayTransit` can chain across multiple VNets, so the reachable set is the transitive peering closure, not just direct neighbours.
- A subnet with a `Microsoft.Storage` or `Microsoft.Sql` service endpoint means a storage account or SQL server may accept connections from that subnet with its network ACL, so a foothold in the subnet bypasses the resource firewall.
- Pair with [network security groups](network-security-groups.md) for what is actually allowed, and with [private endpoints](private-endpoints.md) for PaaS pulled onto the VNet.

## Tools

- **az network vnet** (`list`, `peering list`, `subnet list`): the peering and endpoint map.
- **MicroBurst** / **Azure Resource Graph**: subscription-wide network graph.
- **Stormspotter** / **AzureHound (BARK)**: graph the resource and reachability relationships.

## References

- [HackTricks Cloud: Azure networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: virtual network peering](https://learn.microsoft.com/azure/virtual-network/virtual-network-peering-overview)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
