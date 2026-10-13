---
order: 3
title: "Image poisoning : backdoorer un tag que la compute en aval va exécuter"
description: "Pousser un tag d'image backdooré que ECS, EKS ou Lambda vont récupérer et exécuter sous un rôle privilégié."
keywords:
  - ECR
  - image poisoning
  - supply chain
  - tag
  - backdoor
---

# Image poisoning

Si vous pouvez pousser vers un dépôt ECR, vous pouvez écraser un tag que des charges de travail en cours récupèrent. Les services ECS, les déploiements EKS et les Lambdas basées sur une image de conteneur re-récupèrent l'image à chaque déploiement ou mise à l'échelle, de sorte qu'un `:latest` backdooré (ou un tag épinglé que vous pouvez écraser) exécute votre code sous le rôle détenu par la charge de travail, ce qui constitue à la fois de l'exécution et de la persistance.

## Empoisonner un tag

```bash
aws ecr get-login-password | docker login --username AWS --password-stdin \
  <acct>.dkr.ecr.<region>.amazonaws.com

# Rebuild the real image with an added payload, keep the same tag
docker build -t <acct>.dkr.ecr.<region>.amazonaws.com/<repo>:latest .
docker push <acct>.dkr.ecr.<region>.amazonaws.com/<repo>:latest
```

Forcez la récupération en déclenchant un nouveau déploiement (`ecs update-service --force-new-deployment`) là où vous en avez les droits, ou attendez la prochaine mise à l'échelle ou la prochaine release.

## Notes d'exploitation

- Conservez l'entrypoint d'origine et ajoutez votre payload à côté, pour que le service continue de fonctionner et que le changement ne soit pas repéré sur le seul comportement.
- L'immutabilité des tags, lorsqu'elle est activée, empêche d'écraser un tag existant ; poussez un nouveau tag et attendez un déploiement qui le référence à la place.
- Le payload hérite du rôle de la tâche ou de la fonction, associez-le donc au vol du rôle de tâche décrit dans [containers](../containers.md) pour savoir ce que vous gagnerez.

## Outils

- **docker** / **crane** : reconstruire et pousser l'image empoisonnée.
- **Pacu** (`ecs__backdoor_task_def`) : rediriger une définition de tâche vers votre image.

## Références

- [HackTricks Cloud: ECR image poisoning](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecr-enum.html)
- [AWS: ECR image tag mutability](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
