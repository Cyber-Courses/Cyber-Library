---
title: "Réseau Azure"
order: 7
description: "Attaquer le réseau Azure : network security groups et exposition VNet, private endpoints, DNS takeover, Front Door et CDN, et Bastion."
keywords:
  - réseau Azure
  - NSG
  - VNet
  - DNS takeover
  - private endpoint
---

# Réseau

L'accessibilité réseau d'Azure décide de ce qu'un pied dans la porte peut atteindre. Les network security groups filtrent le trafic, les VNets et les peerings définissent le rayon d'action interne, les private endpoints tirent les services PaaS sur des adresses internes, et les services de périphérie (Front Door, CDN, Bastion) ainsi que les enregistrements DNS sont chacun exploitables à leur manière. L'énumération ici transforme la posture réseau de l'abonnement en liste de cibles et en carte de pivot.

## Ce qui se rattache ici

- **[Network security groups](network-security-groups.md)** : lire et réécrire les règles NSG pour exposer ou atteindre des ressources filtrées.
- **[VNet](vnet.md)** : peering, service endpoints et accès privé pour pivoter entre sous-réseaux.
- **[Private endpoints](private-endpoints.md)** : Private Link pour atteindre des ressources PaaS sur des adresses internes.
- **[DNS takeover](dns-takeover.md)** : revendiquer les enregistrements orphelins de ressources App Service, Traffic Manager, CDN ou IP publiques supprimées.
- **[Front Door et CDN](front-door-and-cdn.md)** : exposition de l'origine, abus du host-header et du routage, et endpoints orphelins.
- **[Bastion](bastion.md)** : pivoter vers des VM privées via RDP et SSH.

## Références

- [HackTricks Cloud : réseau Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
- [Microsoft : documentation réseau Azure](https://learn.microsoft.com/azure/networking/)
