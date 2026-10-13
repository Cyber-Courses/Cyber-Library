---
title: "PassRole"
description: "iam:PassRole vers un service qui s'exécute avec le rôle passé : confier un rôle privilégié à EC2, Lambda, Glue, CloudFormation, Data Pipeline, CodeBuild ou SageMaker."
keywords:
  - PassRole
  - rôle de service
  - EC2
  - Lambda
  - CloudFormation
---

# PassRole

`iam:PassRole` est l'autorisation de confier un rôle IAM à un service AWS. À elle seule, elle ne fait rien ; combinée à une action qui **crée ou met à jour une ressource qui s'exécute avec un rôle**, elle vous permet de passer un rôle plus privilégié que le vôtre à quelque chose qui exécutera votre code ou renverra ses identifiants. Le rôle cible n'a besoin que d'une politique de confiance autorisant le service (par exemple `ec2.amazonaws.com`), ce qui est le cas par défaut lorsque le rôle a été conçu pour ce service.

Le schéma est toujours le même : trouver un rôle privilégié passable (`iam:ListRoles`, ou le graphe issu de l'[énumération](../../enumeration.md)), puis le piloter via l'une des créations de service ci-dessous.

## Cibles

- **[EC2](ec2.md)** : lancer une instance avec un profil d'instance privilégié, puis lire ses identifiants depuis IMDS.
- **[Lambda](lambda.md)** : créer une fonction avec un rôle d'exécution privilégié et l'invoquer.
- **[Glue](glue.md)** : un job ou un dev endpoint Glue s'exécutant en tant que rôle passé.
- **[CloudFormation](cloudformation.md)** : déployer une stack avec un rôle de service passé.
- **[Data Pipeline](data-pipeline.md)** : un pipeline dont les activités s'exécutent en tant que rôle passé.
- **[CodeBuild](codebuild.md)** : un projet de build s'exécutant en tant que rôle de service passé.
- **[SageMaker](sagemaker.md)** : un notebook ou un job d'entraînement s'exécutant en tant que rôle passé.
- **[Step Functions](step-functions.md)** : une machine à états dont les tâches s'exécutent en tant que rôle passé.
- **[CodeStar](codestar.md)** : une chaîne d'outils de projet déployée en tant que rôle passé, ou auto-ajout comme propriétaire de projet.

## Trouver des rôles passables

```bash
aws iam list-roles --query 'Roles[].Arn'
# a role is useful if its trust policy allows the service you can drive
aws iam get-role --role-name <r> --query 'Role.AssumeRolePolicyDocument'
```

## Références

- [Rhino Security Labs: AWS privilege escalation (PassRole paths)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: iam:PassRole](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-iam-privesc.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
