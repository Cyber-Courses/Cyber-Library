---
title: "Organizations"
order: 7
description: "Pivoter à travers une organisation AWS : endosser l'OrganizationAccountAccessRole créé par le compte de gestion dans les comptes membres et manipuler les politiques de contrôle des services."
keywords:
  - AWS Organizations
  - OrganizationAccountAccessRole
  - SCP
  - compte de gestion
  - compte membre
  - inter-comptes
---

# Organizations

AWS Organizations relie de nombreux comptes sous un même compte de gestion. Cette structure est un pivot : le compte de gestion détient un rôle par défaut vers chaque membre, et il définit les politiques de contrôle des services qui les bornent tous. Atteindre le compte de gestion, ou un compte capable d'en endosser d'autres, se répercute en cascade sur tout le parc.

## Pages

- **[Rôle de compte membre](member-account-role.md)** : endosser l'OrganizationAccountAccessRole par défaut que le compte de gestion crée dans chaque membre.
- **[Manipulation de SCP](scp-manipulation.md)** : modifier les politiques de contrôle des services pour lever les garde-fous à l'échelle de l'organisation.

## Références

- [HackTricks Cloud: AWS Organizations](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-organizations-enum.html)
- [AWS: Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
