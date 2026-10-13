---
title: "SageMaker : voler données, modèles et rôle de notebook"
order: 3
description: "Abuser des notebooks, jobs d'entraînement et endpoints SageMaker pour voler des données, des modèles et le rôle attaché."
keywords:
  - SageMaker
  - notebook
  - job d'entraînement
  - modèle
  - rôle
---

# SageMaker

SageMaker exécute des notebooks, des jobs d'entraînement et des endpoints d'inférence, chacun attaché à un rôle d'exécution ayant accès aux données d'entraînement dans S3. Deux choses valent la peine d'être prises : les **données et modèles**, et le **rôle d'exécution**. Un principal disposant de `sagemaker:CreatePresignedNotebookInstanceUrl` ouvre un shell dans un notebook en service et lit les identifiants de son rôle depuis le service de métadonnées, ce qui est la même élévation `PassRole` documentée dans [identity](../identity/privilege-escalation/pass-role/sagemaker.md).

## Ouvrir un notebook que vous pouvez atteindre

```bash
aws sagemaker list-notebook-instances
aws sagemaker create-presigned-notebook-instance-url --notebook-instance-name <nb>
# open the URL, drop to a terminal, then read the attached role from IMDS
```

## Lire données et modèles

```bash
aws sagemaker list-training-jobs
aws sagemaker describe-training-job --training-job-name <j> \
  --query 'InputDataConfig[].DataSource.S3DataSource.S3Uri'   # training data in S3
aws sagemaker list-models ; aws sagemaker describe-model --model-name <m>
```

## Notes d'exploitation

- Le rôle d'exécution du notebook est généralement large (S3 complet, parfois davantage) ; le voler depuis IMDS transforme l'accès aux données en mouvement dans le compte.
- Les définitions de jobs d'entraînement et de modèles pointent directement vers les compartiments S3 contenant les données, que vous lisez ensuite avec le rôle que vous venez de prendre.
- Créer un nouveau notebook avec un rôle privilégié passé est la variante plus bruyante de `PassRole` lorsque vous ne pouvez pas atteindre un notebook existant.

## Outils

- **AWS CLI** (`sagemaker create-presigned-notebook-instance-url`, `describe-training-job`).
- **Pacu** (`sagemaker__*`) : énumérer notebooks, jobs et modèles.

## Références

- [HackTricks Cloud: AWS SageMaker](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: SageMaker notebook instance security](https://docs.aws.amazon.com/sagemaker/latest/dg/security.html)
- [CloudFox: enumerating roles and data behind compute services](https://github.com/BishopFox/cloudfox)
- [Datadog Security Labs: AWS attack research](https://securitylabs.datadoghq.com/)
