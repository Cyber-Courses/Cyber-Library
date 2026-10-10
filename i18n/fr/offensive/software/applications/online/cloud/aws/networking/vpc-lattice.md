---
title: "VPC Lattice : atteindre des services entre VPC via les auth policies"
order: 7
description: "Détourner les service networks et auth policies de VPC Lattice pour atteindre des services entre VPC et comptes."
keywords:
  - VPC Lattice
  - service network
  - auth policy
  - cross-account
  - service mesh
---

# VPC Lattice

VPC Lattice connecte des services entre VPC et comptes via un service network, avec un accès régi par des auth policies de style IAM plutôt que par le routage réseau. Une auth policy permissive (un principal avec wildcard, ou aucune policy du tout) permet à tout principal capable d'atteindre le service network d'appeler les services enregistrés, transformant Lattice en pivot inter-comptes au niveau applicatif.

## Énumérer le maillage

```bash
aws vpc-lattice list-service-networks
aws vpc-lattice list-services
aws vpc-lattice list-service-network-service-associations --service-network-identifier <id>
aws vpc-lattice get-auth-policy --resource-identifier <service-or-network-arn>
```

## Notes d'exploitation

- Une auth policy avec `"Principal":"*"` ou `AWS:"*"` et sans condition expose le service à tout appelant authentifié sur le réseau.
- Lattice contourne les frontières traditionnelles de security group et de sous-réseau, de sorte qu'un service inaccessible par le routage peut tout de même être appelable via le service network.
- Les associations de services entre comptes font d'une policy permissive une portée inter-comptes, et pas seulement intra-compte.

## Outils

- **AWS CLI** (`vpc-lattice ...`) : énumération des service networks, services et auth policies.
- **ScoutSuite** / **Prowler** : signaler les auth policies Lattice permissives.

## Références

- [AWS : auth policies de VPC Lattice](https://docs.aws.amazon.com/vpc-lattice/latest/ug/auth-policies.html)
- [HackTricks Cloud : services réseau AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [CloudFox : cartographier les chemins réseau cloud accessibles](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
