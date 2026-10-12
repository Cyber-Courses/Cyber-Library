---
title: "Direct Connect : relier des réseaux on-premises au VPC"
order: 8
description: "Abuser des liaisons Direct Connect et des interfaces virtuelles qui relient des réseaux on-premises au VPC."
keywords:
  - Direct Connect
  - virtual interface
  - hybrid
  - on-premises
  - VPC
---

# Direct Connect

Direct Connect est une liaison dédiée entre un réseau on-premises et AWS, exposée via des interfaces virtuelles (VIF) qui se rattachent à des VPC ou à une passerelle Direct Connect. Un point d'appui de part et d'autre d'une VIF atteint l'autre côté : un rôle VPC compromis lit le routage hybride qui mène vers l'on-premises, et un hôte on-premises compromis emprunte la VIF privée pour entrer dans les plages VPC qui supposent la liaison digne de confiance.

## Énumérer les liaisons et les interfaces

```bash
aws directconnect describe-connections
aws directconnect describe-virtual-interfaces \
  --query "virtualInterfaces[].[virtualInterfaceId,virtualInterfaceType,vlan,amazonAddress,customerAddress,virtualGatewayId]"
aws directconnect describe-direct-connect-gateways
```

## Notes d'exploitation

- Les paires d'adresses de VIF privée (`amazonAddress` / `customerAddress`) révèlent les plages internes de la liaison hybride à scanner depuis un point d'appui VPC.
- Le trafic sur une VIF privée est souvent implicitement digne de confiance des deux côtés : les contrôles qui supposent une frontière internet ne s'appliquent donc pas.
- Une passerelle Direct Connect peut rattacher la liaison à plusieurs VPC et comptes, élargissant le rayon d'impact de la compromission de l'un ou l'autre côté.

## Outils

- **AWS CLI** (`directconnect describe-*`) : énumération des liaisons, VIF et passerelles.
- **Analyse de routes/VPC** ([VPC](vpc.md)) : corréler les plages annoncées par les VIF avec les sous-réseaux joignables.

## Références

- [AWS : interfaces virtuelles Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/WorkingWithVirtualInterfaces.html)
- [HackTricks Cloud : réseau hybride AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [CloudFox : cartographier les chemins réseau cloud joignables](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
