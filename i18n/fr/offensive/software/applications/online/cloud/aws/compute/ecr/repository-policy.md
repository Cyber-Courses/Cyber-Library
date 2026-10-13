---
order: 2
title: "Politique de dépôt : récupération et poussée inter-comptes"
description: "Abuser d'une politique de dépôt ECR permissive pour récupérer ou pousser des images par-delà les frontières de compte."
keywords:
  - ECR
  - politique de dépôt
  - inter-comptes
  - poussée
  - récupération
---

# Politique de dépôt

Une **politique de dépôt** ECR est une politique de ressource qui peut accorder à d'autres principaux, y compris des comptes externes entiers, le droit de récupérer ou de pousser. Une politique qui autorise `ecr:*` ou désigne un principal trop large laisse un attaquant d'un compte lire des images d'un dépôt d'un autre compte, ou y pousser des images, transformant une politique trop permissive en accès inter-comptes aux données ou en écriture sur la chaîne d'approvisionnement.

## Lire et abuser de la politique

```bash
aws ecr get-repository-policy --repository-name <r> --query policyText --output text

# If pull is allowed cross-account: authenticate to the owning registry and pull
aws ecr get-login-password --region <region> | docker login --username AWS \
  --password-stdin <owner-acct>.dkr.ecr.<region>.amazonaws.com
docker pull <owner-acct>.dkr.ecr.<region>.amazonaws.com/<r>:<tag>
```

Si la poussée est autorisée, le dépôt est une cible d'[image poisoning](image-poisoning.md).

## Notes d'exploitation

- Un `Principal` à `*` ou le root d'un autre compte sur une action de récupération expose toutes les images du dépôt en inter-comptes.
- Les droits de poussée en inter-comptes sont le cas dangereux : ils permettent de planter un tag que le pipeline de déploiement du propriétaire exécutera.
- Combinez avec l'[énumération](../../identity/enumeration.md) pour trouver quels dépôts les politiques exposent réellement, puis ciblez ceux qui alimentent des charges de travail privilégiées.

## Outils

- **AWS CLI** (`get-repository-policy`, `ecr get-login-password`) : lire la politique, s'authentifier.
- **docker** / **crane** : récupérer ou pousser au-delà de la frontière.

## Références

- [AWS: ECR repository policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-policies.html)
- [HackTricks Cloud: ECR](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecr-enum.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
