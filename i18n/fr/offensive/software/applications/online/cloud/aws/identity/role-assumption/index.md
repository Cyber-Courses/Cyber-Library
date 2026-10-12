---
title: "Assomption de rôle"
order: 3
description: "Assumer des rôles IAM AWS via sts:AssumeRole : accès cross-account et abus du confused deputy sur des politiques de confiance trop larges et des external ID manquants."
keywords:
  - AssumeRole
  - STS
  - cross-account
  - confused deputy
  - trust policy
---

# Assomption de rôle

Un rôle est assumable par quiconque sa **politique de confiance** autorise. `sts:AssumeRole` renvoie des identifiants temporaires pour le rôle et, parce que les politiques de confiance sont fréquemment rédigées de façon trop large (un compte entier comme principal, un joker, ou un compte tiers sans external ID), l'assomption de rôle est à la fois une primitive de mouvement latéral et d'escalade.

## Assumer un rôle

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::<acct>:role/<role> \
  --role-session-name s --query Credentials
# export AccessKeyId / SecretAccessKey / SessionToken and continue as the role
```

## Ce qui est regroupé ici

- **[Cross-account](cross-account.md)** : les politiques de confiance qui nomment un autre compte (ou `root`), laissant tout principal de ce compte assumer.
- **[Confused deputy](confused-deputy.md)** : des rôles de fournisseur tiers assumables sans l'external ID censé les restreindre.

## Trouver des rôles assumables

Lisez les documents de confiance depuis le dump d'[énumération](../enumeration.md) et recherchez les valeurs `Principal` plus larges qu'un seul ARN de rôle, ainsi que les instructions `sts:AssumeRole` dépourvues de `Condition` sur `sts:ExternalId`.

## Références

- [HackTricks Cloud : AssumeRole et politiques de confiance](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [AWS : le problème du confused deputy](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Stratus Red Team : techniques AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
