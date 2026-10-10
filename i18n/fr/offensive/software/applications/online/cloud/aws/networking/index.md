---
title: "Réseau AWS"
order: 7
description: "Détourner le réseau AWS pour atteindre et exposer des ressources : security groups trop permissifs, services exposés sur Internet, accessibilité et peering de VPC, Route53 et CloudFront, prise de contrôle par DNS orphelin, et les services edge et hybrides qui élargissent l'accès."
keywords:
  - security group
  - VPC
  - Route53
  - dangling DNS
  - networking
---

# Réseau

Le réseau AWS détermine ce qui est accessible : des **security groups** trop ouverts, des services liés à Internet, le peering et les endpoints de **VPC** qui permettent à un point d'appui dans un réseau d'en atteindre un autre, et les services DNS et edge qui mettent en frontal et acheminent le trafic. Le réseau compromet rarement un compte à lui seul, mais c'est par lui qu'un point d'appui accessible se transforme en portée sur l'ensemble de l'environnement, et qu'un nom de confiance est capturé.

## Ce qui est regroupé ici

- **[Security groups](security-groups.md)** : ports de management et services exposés sur Internet du fait d'un ingress trop permissif.
- **[VPC](vpc.md)** : peering, endpoints et routage pour pivoter entre réseaux et comptes.
- **[Route53](route53.md)** : détournement d'enregistrements et empoisonnement de la résolution là où vous disposez d'un accès en écriture à la zone.
- **[CloudFront](cloudfront.md)** : divulgation d'origine et abus du comportement de cache pour atteindre des origines protégées.
- **[Prise de contrôle par DNS orphelin](dangling-dns-takeover.md)** : revendiquer la ressource S3, CloudFront ou ELB orpheline derrière un enregistrement périmé.
- **[Global Accelerator](global-accelerator.md)** : listeners et endpoint groups pour mettre en frontal des endpoints derrière des IP anycast de confiance.
- **[VPC Lattice](vpc-lattice.md)** : auth policies de service network permissives pour une portée inter-VPC et inter-comptes.
- **[Direct Connect](direct-connect.md)** : interfaces virtuelles qui relient les réseaux on-premises au VPC.
- **[Network Firewall](network-firewall.md)** : lacunes dans les rule groups et démantèlement de l'inspection.
- **[WAF](waf.md)** : contournement de web ACL par l'encodage, les limites de taille et les requêtes directes vers l'origine.

Le déplacement latéral au niveau réseau se trouve ici ; le déplacement inter-comptes basé sur l'identité via `AssumeRole` se trouve dans [identité](../identity/index.md).

## Références

- [HackTricks Cloud : VPC et réseau AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [AWS : security groups pour votre VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [can-i-take-over-xyz (empreintes de prise de contrôle de sous-domaine)](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
