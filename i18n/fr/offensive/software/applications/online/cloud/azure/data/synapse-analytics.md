---
title: "Synapse Analytics : pools, pipelines et l'identité du workspace"
order: 4
description: "Attaquer Azure Synapse : pools SQL et Spark, pipelines et identité managée du workspace."
keywords:
  - Synapse
  - SQL pool
  - Spark
  - pipeline
  - identité managée
---

# Synapse Analytics

Azure Synapse réunit des pools SQL, des pools Spark et des pipelines de type Data Factory dans un seul workspace, et ce workspace porte une **identité managée**. Atteindre un workspace Synapse donne à la fois les données contenues dans ses pools et un moyen d'exécuter du code (notebooks Spark, activités de pipeline) sous l'identité du workspace, laquelle dispose en général d'un accès au compte de stockage principal du workspace, et souvent davantage.

## Atteindre le workspace

```bash
az synapse workspace list -o table
# firewall rule to reach the SQL endpoint
az synapse workspace firewall-rule create --workspace-name <ws> -g <rg> \
  --name open --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
```

## S'exécuter sous l'identité du workspace

```bash
# a Spark notebook or a pipeline Web activity runs as the workspace MI
az synapse spark session create --workspace-name <ws> --spark-pool-name <pool> \
  --name s --executor-size Small --executors 2
# or trigger a pipeline that calls ARM/Graph/storage as the identity
az synapse pipeline create-run --workspace-name <ws> --name <pipeline>
```

## Notes d'exploitation

- L'identité du workspace possède par défaut le rôle `Storage Blob Data Contributor` sur le compte ADLS principal, donc contrôler le workspace revient à lire et écrire sur ce data lake.
- Un pool SQL serverless peut lire tout stockage que l'identité du workspace ou le principal qui exécute la requête peut atteindre avec `OPENROWSET`, un chemin rapide vers les données blob.
- Les notebooks Spark offrent une exécution de code arbitraire à l'intérieur du workspace, utile pour voler le token depuis le point de terminaison de l'identité.

## Outils

- **az cli** (`az synapse spark session create`, `az synapse pipeline create-run`).
- **MicroBurst** : énumération du workspace et de l'identité.

## Références

- [Microsoft : identité managée Synapse](https://learn.microsoft.com/azure/synapse-analytics/security/synapse-workspace-managed-identity)
- [HackTricks Cloud : Azure Synapse](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps : chemins d'attaque des identités managées Azure (BARK)](https://github.com/BloodHoundAD/BARK)
