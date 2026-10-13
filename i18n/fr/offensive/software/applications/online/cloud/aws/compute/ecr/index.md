---
order: 4
title: "ECR"
description: "Attaquer Elastic Container Registry : récupérer des images depuis des dépôts publics ou exposés par une politique, et empoisonner les images que la compute en aval va exécuter."
keywords:
  - ECR
  - registre de conteneurs
  - politique de dépôt
  - image poisoning
  - dépôt public
---

# ECR

Elastic Container Registry stocke les images que ECS, EKS, Lambda et App Runner récupèrent et exécutent. Cela en fait une cible à double tranchant : les images sont une **source de secrets** (credentials et détails internes intégrés dans les couches), et un dépôt accessible en écriture est un **point d'ancrage sur la chaîne d'approvisionnement**, car un tag empoisonné s'exécute au prochain déploiement sous le rôle détenu par la charge de travail.

## Ce qui s'y rattache

- **[Dépôt public](public-repository.md)** : récupérer depuis des dépôts publics ou faiblement cadrés pour en extraire des secrets.
- **[Politique de dépôt](repository-policy.md)** : récupération et poussée inter-comptes via une politique de dépôt permissive.
- **[Image poisoning](image-poisoning.md)** : pousser un tag backdooré que la compute en aval va exécuter.

## S'authentifier auprès d'ECR

```bash
aws ecr get-login-password | docker login --username AWS --password-stdin \
  <acct>.dkr.ecr.<region>.amazonaws.com
aws ecr describe-repositories ; aws ecr list-images --repository-name <r>
```

## Références

- [HackTricks Cloud: ECR](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecr-enum.html)
- [AWS: ECR repository policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-policies.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
