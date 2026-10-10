---
title: "VNet : le peering et les service endpoints comme pivot"
order: 2
description: "Détourner les réseaux virtuels Azure : peering, service endpoints et accès privé pour pivoter entre sous-réseaux et ressources."
keywords:
  - VNet
  - peering
  - service endpoint
  - subnet
  - pivot
---

# VNet

Un réseau virtuel constitue le rayon d'impact interne. Depuis une VM ou un conteneur compromis, le **peering** de VNet dépasse souvent largement le sous-réseau local, car les peerings sont mis en place par commodité et rarement segmentés, et les **service endpoints** étendent l'identité d'un sous-réseau aux ressources PaaS (stockage, SQL) qui font alors confiance au trafic provenant du sous-réseau. Cartographier le graphe de peering indique jusqu'où une seule prise de pied peut atteindre.

## Cartographier le réseau accessible

```bash
# the VNets, their peerings, and subnets
az network vnet list --query "[].{Name:name,RG:resourceGroup,Space:addressSpace.addressPrefixes}" -o table
az network vnet peering list --vnet-name <vnet> -g <rg> \
  --query "[].{Name:name,Remote:remoteVirtualNetwork.id,State:peeringState}" -o table
# subnets and the service endpoints they carry
az network vnet subnet list --vnet-name <vnet> -g <rg> \
  --query "[].{Name:name,Endpoints:serviceEndpoints[].service}" -o table
```

## Notes d'exploitation

- Un peering `Connected` avec `allowForwardedTraffic`/`allowGatewayTransit` peut s'enchaîner sur plusieurs VNets, si bien que l'ensemble accessible correspond à la fermeture transitive du peering, et non aux seuls voisins directs.
- Un sous-réseau doté d'un service endpoint `Microsoft.Storage` ou `Microsoft.Sql` signifie qu'un compte de stockage ou un serveur SQL peut accepter les connexions depuis ce sous-réseau via sa network ACL : une prise de pied dans le sous-réseau contourne alors le pare-feu de la ressource.
- À associer aux [network security groups](network-security-groups.md) pour ce qui est réellement autorisé, et aux [private endpoints](private-endpoints.md) pour les ressources PaaS rattachées au VNet.

## Outils

- **az network vnet** (`list`, `peering list`, `subnet list`) : la carte du peering et des endpoints.
- **MicroBurst** / **Azure Resource Graph** : graphe réseau à l'échelle de l'abonnement.
- **Stormspotter** / **AzureHound (BARK)** : graphe des relations de ressources et d'accessibilité.

## Références

- [HackTricks Cloud: Azure networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: virtual network peering](https://learn.microsoft.com/azure/virtual-network/virtual-network-peering-overview)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
