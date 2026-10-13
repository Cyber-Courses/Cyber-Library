---
title: "Rôle de compte membre : endosser OrganizationAccountAccessRole"
description: "sts:AssumeRole vers l'OrganizationAccountAccessRole que le compte de gestion crée par défaut dans chaque compte membre."
keywords:
  - OrganizationAccountAccessRole
  - compte membre
  - AssumeRole
  - compte de gestion
  - Organizations
---

# Rôle de compte membre

Lorsqu'un compte est créé dans une organisation ou invité à la rejoindre, AWS provisionne dans le membre un rôle nommé **OrganizationAccountAccessRole**, qui fait confiance au compte de gestion avec des autorisations d'administrateur. Depuis le compte de gestion (ou tout principal y étant autorisé à l'endosser), vous accédez à n'importe quel compte membre en tant qu'administrateur.

## Endosser vers un membre

```bash
aws organizations list-accounts --query 'Accounts[].Id'
aws sts assume-role \
  --role-arn arn:aws:iam::<member-acct>:role/OrganizationAccountAccessRole \
  --role-session-name s
```

## Notes d'exploitation

- Le nom de rôle est celui par défaut ; certaines organisations le renomment, alors énumérez les rôles dans un membre si le nom par défaut est absent.
- Cela fait du compte de gestion la cible de plus haute valeur dans une organisation : il est administrateur partout par conception.
- Les comptes créés par l'organisation disposent du rôle automatiquement ; les comptes invités peuvent ne pas l'avoir, de sorte que la couverture varie.

## Outils

- **AWS CLI** (`organizations list-accounts`, `sts assume-role`).
- **PMapper** / **Pacu** : découverte de chemins inter-comptes.

## Références

- [AWS: OrganizationAccountAccessRole](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html)
- [HackTricks Cloud: AWS Organizations](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-organizations-enum.html)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
