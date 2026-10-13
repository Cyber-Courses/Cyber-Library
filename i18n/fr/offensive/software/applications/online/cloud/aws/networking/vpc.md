---
title: "VPC : pivoter via le peering, les endpoints et le routage"
order: 2
description: "Pivoter via le peering de VPC, les endpoints et le routage pour atteindre des services internes et d'autres comptes."
keywords:
  - VPC
  - peering
  - endpoint
  - routing
  - pivot
---

# VPC

Une fois que vous détenez une instance ou un rôle à l'intérieur d'un VPC, la topologie réseau détermine jusqu'où vous pouvez aller. Les connexions de peering, les transit gateways et les tables de routage relient les VPC entre eux (souvent entre comptes), et les endpoints de VPC exposent des chemins d'accès aux services AWS. Lire le graphe de routage et de peering avec `ec2:Describe*` révèle quelles plages internes et quels comptes sont accessibles depuis votre point d'appui.

## Lire le graphe d'accessibilité

```bash
aws ec2 describe-vpc-peering-connections \
  --query "VpcPeeringConnections[].[VpcPeeringConnectionId,RequesterVpcInfo.CidrBlock,AccepterVpcInfo.CidrBlock]"
aws ec2 describe-route-tables \
  --query "RouteTables[].Routes[].[DestinationCidrBlock,GatewayId,VpcPeeringConnectionId,TransitGatewayId]"
aws ec2 describe-transit-gateway-attachments
```

## Endpoints et services d'interface

```bash
# interface/gateway endpoints reveal which services the VPC can reach privately
aws ec2 describe-vpc-endpoints \
  --query "VpcEndpoints[].[ServiceName,VpcEndpointType,VpcId]"
```

## Notes d'exploitation

- Une route de peering vers le CIDR d'un autre compte est un pivot inter-comptes : scannez cette plage depuis votre point d'appui dès lors qu'une route existe.
- Les transit gateways concentrent l'accessibilité : un seul attachement peut exposer de nombreux VPC d'un coup.
- Les endpoints d'interface dotés de politiques d'endpoint permissives permettent à un principal situé dans le VPC d'atteindre des services que le compte entendait garder privés.

## Outils

- **AWS CLI** (`ec2 describe-*`) : le graphe de routage et de peering.
- **ScoutSuite** : visualiser l'exposition du peering et des endpoints.
- **pmapper** : les arêtes IAM inter-comptes, qui recoupent souvent le peering réseau.

## Références

- [HackTricks Cloud : AWS VPC et réseau](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [AWS : VPC peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
- [CloudFox : cartographier les chemins réseau cloud accessibles](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
