---
order: 3
title: "Compute AWS"
description: "Attaquer la compute AWS : user data et profils d'instance EC2, SSM run command et Session Manager, conteneurs ECS et EKS et images ECR, et les runtimes Elastic Beanstalk, Image Builder, App Runner, Batch et Lightsail."
keywords:
  - EC2
  - SSM
  - ECR
  - conteneurs
  - profil d'instance
---

# Compute

La compute dans AWS est l'endroit où IAM rencontre l'exécution de code. Presque chaque service de compute s'exécute avec un **rôle** attaché, de sorte que prendre le contrôle d'une charge de travail, ou en lancer une que vous contrôlez, livre les credentials de ce rôle et tout ce qu'il peut atteindre. Les mêmes services détiennent aussi des secrets au repos : scripts user-data, snapshots, images et configuration d'environnement. Les pages ici travaillent sur les deux fronts, voler le rôle et lire les données.

## Surfaces

- **[EC2](ec2/index.md)** : secrets et code du user data, rôle du profil d'instance, et récupération de données depuis les snapshots EBS et les AMI partagées.
- **[SSM](ssm/index.md)** : Systems Manager run command et shells Session Manager sur des instances managées, sous le rôle d'instance.
- **[Conteneurs](containers.md)** : vol du rôle de tâche ECS et EKS, abus de définition de tâche, et runtime de conteneur.
- **[ECR](ecr/index.md)** : récupérer depuis des dépôts exposés et empoisonner les images que la compute en aval va exécuter.
- **[Elastic Beanstalk](elastic-beanstalk.md)** : environnements, leur rôle de profil d'instance, et secrets dans la configuration.
- **[Image Builder](image-builder.md)** : pipelines et composants qui intègrent du code dans les AMI de référence.
- **[App Runner](app-runner.md)** : services et leur rôle d'instance, et déploiement par source ou par image.
- **[Batch](batch.md)** : définitions de jobs et environnements de compute exécutant des conteneurs sous un rôle de job.
- **[Lightsail](lightsail.md)** : instances, clés et snapshots qui restent hors du VPC principal et de la visibilité IAM.

## Références

- [HackTricks Cloud: AWS EC2, SSM and compute](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Rhino Security Labs: AWS privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
