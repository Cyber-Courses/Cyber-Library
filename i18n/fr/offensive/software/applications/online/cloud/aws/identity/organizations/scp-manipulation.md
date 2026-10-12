---
title: "Manipulation de SCP : lever les garde-fous à l'échelle de l'organisation"
description: "Modifier les politiques de contrôle des services depuis le compte de gestion pour lever les garde-fous à travers toute l'organisation."
keywords:
  - SCP
  - politique de contrôle des services
  - Organizations
  - compte de gestion
  - garde-fou
---

# Manipulation de SCP

Les politiques de contrôle des services constituent le plafond d'autorisations à l'échelle de l'organisation : elles limitent ce que les principaux des comptes membres peuvent faire, indépendamment de leurs politiques IAM. Depuis le compte de gestion (ou un administrateur délégué), modifier ou détacher une SCP retire ce plafond, débloquant des actions que l'organisation comptait sur les SCP pour refuser.

## Assouplir un garde-fou

```bash
aws organizations list-policies --filter SERVICE_CONTROL_POLICY
aws organizations update-policy --policy-id <scp-id> \
  --content '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
# or detach a restrictive SCP from an OU or account
aws organizations detach-policy --policy-id <scp-id> --target-id <ou-or-account>
```

## Notes d'exploitation

- Les SCP n'accordent pas d'autorisations, elles les bornent : lever une SCP ne compte donc qu'en présence d'autorisations IAM dans le compte cible, mais cela supprime un contrôle que les équipes supposent en place.
- Cela requiert un accès au compte de gestion ; c'est une manœuvre post-compromission une fois la racine de l'organisation en main, souvent associée au [rôle de compte membre](member-account-role.md).
- Modifier la base FullAWSAccess ou attacher une politique « tout autoriser » est le changement le plus large et le plus visible.

## Outils

- **AWS CLI** (`organizations update-policy` / `detach-policy`).

## Références

- [AWS: service control policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [HackTricks Cloud: AWS Organizations](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-organizations-enum.html)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
