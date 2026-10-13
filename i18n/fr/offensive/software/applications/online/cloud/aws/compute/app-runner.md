---
order: 7
title: "App Runner : abus du rôle d'instance et du déploiement"
description: "Abuser des services App Runner et de leur rôle d'instance, ainsi que du déploiement depuis une source ou une image pour exécuter du code attaquant."
keywords:
  - App Runner
  - rôle d'instance
  - service
  - image
  - déploiement
---

# App Runner

App Runner est un runtime managé pour conteneurs et applications web. Un service s'exécute avec un **rôle d'instance** (le rôle que le code de votre application assume) et, en option, un rôle d'accès pour tirer depuis ECR. Créer ou mettre à jour un service avec votre image ou votre source exécute votre code avec ce rôle d'instance, et déployer depuis un dépôt source connecté constitue une voie de chaîne d'approvisionnement.

## Déployer votre code

```bash
aws apprunner list-services
aws apprunner describe-service --service-arn <arn> \
  --query 'Service.InstanceConfiguration.InstanceRoleArn'

# Point a service at your image to run code under its instance role
aws apprunner create-service --service-name x \
  --source-configuration 'ImageRepository={ImageIdentifier=<acct>.dkr.ecr.<region>.amazonaws.com/you:latest,ImageRepositoryType=ECR}' \
  --instance-configuration 'InstanceRoleArn=<privileged-role>'
```

## Notes d'exploitation

- Créer un service avec un rôle d'instance choisi nécessite `iam:PassRole`, c'est donc aussi une voie d'élévation de privilèges d'identité.
- Le conteneur en cours d'exécution lit son rôle depuis le point de terminaison d'identifiants d'App Runner, selon le même schéma que les rôles de tâche ECS dans [containers](containers.md).
- Mettre à jour un service existant vers votre image est plus discret que d'en créer un nouveau et hérite du rôle existant.

## Outils

- **AWS CLI** (`apprunner create-service`, `update-service`, `describe-service`).

## Références

- [AWS: App Runner instance roles](https://docs.aws.amazon.com/apprunner/latest/dg/security-iam-roles.html)
- [HackTricks Cloud: AWS services](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
