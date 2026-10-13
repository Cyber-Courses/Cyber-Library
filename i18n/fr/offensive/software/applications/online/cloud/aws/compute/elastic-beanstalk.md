---
order: 5
title: "Elastic Beanstalk : rôle d'environnement et secrets de configuration"
description: "Abuser des environnements Elastic Beanstalk et de leur rôle de profil d'instance, ainsi que des secrets exposés dans la configuration d'environnement."
keywords:
  - Elastic Beanstalk
  - profil d'instance
  - environnement
  - S3
  - déploiement
---

# Elastic Beanstalk

Elastic Beanstalk regroupe EC2, un bucket S3 d'application et un profil d'instance dans un environnement managé. La plateforme expose deux éléments qu'il vaut la peine de prendre : la **configuration d'environnement**, qui porte souvent des secrets dans les propriétés d'environnement, et le **rôle du profil d'instance** sur les hôtes EC2 sous-jacents, atteignable dès que vous exécutez du code dessus.

## Lire la configuration et déployer du code

```bash
aws elasticbeanstalk describe-configuration-settings --application-name <a> --environment-name <e> \
  --query 'ConfigurationSettings[].OptionSettings' | grep -iE 'env|secret|password'

# Application versions live in an S3 bucket; a new version deploys your code
aws elasticbeanstalk describe-applications
aws s3 cp app.zip s3://elasticbeanstalk-<region>-<acct>/<path>
aws elasticbeanstalk create-application-version --application-name <a> \
  --version-label x --source-bundle S3Bucket=...,S3Key=...
aws elasticbeanstalk update-environment --environment-name <e> --version-label x
```

## Notes d'exploitation

- Les propriétés d'environnement sont le magasin de secrets habituel des applications Beanstalk ; extrayez-les en premier.
- Déployer une version d'application exécute votre code sur les hôtes EC2 de l'environnement, ce qui livre le rôle du profil d'instance via IMDS.
- Le bucket S3 d'application lui-même est souvent faiblement cadré ; lisez-le directement lorsque l'API est refusée.

## Outils

- **AWS CLI** (`elasticbeanstalk describe-configuration-settings`, `create-application-version`).

## Références

- [HackTricks Cloud: Elastic Beanstalk](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-elastic-beanstalk-enum.html)
- [AWS: Elastic Beanstalk environment configuration](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
