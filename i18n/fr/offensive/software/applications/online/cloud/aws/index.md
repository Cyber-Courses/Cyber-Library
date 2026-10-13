---
title: "AWS"
description: "Techniques offensives contre Amazon Web Services, décomposées par surface : identité, identifiants, et les classes calcul, stockage, serverless, données, réseau, journalisation et messagerie, l'énumération étant intégrée à chacune. Les attaques se concentrent sur le graphe IAM et l'API AWS, atteints par des clés ou des rôles plutôt que par des exploits d'hôte."
keywords:
  - AWS
  - IAM
  - IMDS
  - S3
  - privilege escalation
---

# AWS

Amazon Web Services se pilote entièrement via son **API**, authentifiée par des **clés d'accès** IAM ou des **identifiants de rôle** temporaires. Presque toute attaque est une question IAM : quel principal est-ce que je détiens, que peut-il faire, vers quoi peut-il s'élever, et quelles ressources peut-il atteindre. Les surfaces ci-dessous suivent cette forme, l'**élévation de privilèges** vivant au sein de l'[identité](identity/index.md) et le vol de données dans le [stockage](storage/index.md) et les [données](data/index.md).

## Énumération

L'énumération n'est pas une surface distincte : chaque surface ci-dessous couvre l'énumération de ses propres ressources, et la découverte non authentifiée (buckets publics, services exposés, identifiants de compte) vit dans la surface qu'elle vise. Ce qui s'étend à travers les surfaces, c'est l'**inventaire à l'échelle du compte**, exécuté d'abord avec **ScoutSuite** et **Prowler**, le graphe IAM cartographié avec **Pacu** et **PMapper**, et les droits du principal courant établis avec `aws sts get-caller-identity` et l'énumération des permissions. Ceux-ci alimentent chaque surface ci-dessous.

## Surfaces

- **[Identité](identity/index.md)** : le graphe IAM, l'assomption de rôle et `PassRole`, l'abus de politiques, la fédération et les voies d'élévation de privilèges, plus les couches d'identité Cognito, Identity Center et Organizations.
- **[Identifiants](credentials/index.md)** : les clés d'accès, les jetons de session STS, les métadonnées d'instance (IMDS), les coffres à secrets et les brokers d'identifiants de service.
- **[Calcul](compute/index.md)** : les user data et profils d'instance EC2, SSM, les conteneurs et ECR, et les runtimes Beanstalk, Image Builder, App Runner, Batch et Lightsail.
- **[Stockage](storage/index.md)** : l'exposition des buckets et objets S3, la récupération EBS et EFS, FSx et Storage Gateway, et AWS Backup.
- **[Serverless](serverless/index.md)** : les rôles d'exécution et le code Lambda, API Gateway, les Function URLs, Step Functions et EventBridge.
- **[Données](data/index.md)** : RDS, DynamoDB, Redshift, SageMaker, Glue, Athena, Lake Formation et les autres banques de données managées.
- **[Réseau](networking/index.md)** : les security groups et la joignabilité VPC, Route53 et CloudFront, la prise de contrôle de DNS orphelin et les services d'edge.
- **[Journalisation et détection](logging-and-detection/index.md)** : désactiver, détourner et contourner CloudTrail, GuardDuty, Config et CloudWatch.
- **[Messagerie](messaging/index.md)** : l'envoi de mail et le phishing SES, et l'abus des topics et files SNS et SQS.

## Références

- [HackTricks Cloud : AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Rhino Security Labs : méthodes d'élévation de privilèges IAM AWS](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [PayloadsAllTheThings : AWS](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cloud%20-%20AWS%20Pentest.md)
