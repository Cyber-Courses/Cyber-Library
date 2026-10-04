---
title: "Synapse Analytics: pools, pipelines, and the workspace identity"
description: "Attacking Azure Synapse: SQL and Spark pools, pipelines, and the workspace managed identity."
keywords:
  - Synapse
  - SQL pool
  - Spark
  - pipeline
  - managed identity
---

# Synapse Analytics

Azure Synapse combines SQL pools, Spark pools, and Data Factory-style pipelines in one workspace, and the workspace carries a **managed identity**. Reaching a Synapse workspace gives both the data in its pools and a way to run code (Spark notebooks, pipeline activities) as the workspace identity, which is usually granted access to the workspace's primary storage account and often more.

## Reaching the workspace

```bash
az synapse workspace list -o table
# firewall rule to reach the SQL endpoint
az synapse workspace firewall-rule create --workspace-name <ws> -g <rg> \
  --name open --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
```

## Running as the workspace identity

```bash
# a Spark notebook or a pipeline Web activity runs as the workspace MI
az synapse spark session create --workspace-name <ws> --spark-pool-name <pool> \
  --name s --executor-size Small --executors 2
# or trigger a pipeline that calls ARM/Graph/storage as the identity
az synapse pipeline create-run --workspace-name <ws> --name <pipeline>
```

## Exploitation notes

- The workspace identity has `Storage Blob Data Contributor` on the primary ADLS account by default, so workspace control is read and write over that data lake.
- A serverless SQL pool can read any storage the workspace identity or the querying principal can reach with `OPENROWSET`, a quick path to blob data.
- Spark notebooks give arbitrary code execution inside the workspace, useful for token theft from the identity endpoint.

## Tools

- **az cli** (`az synapse spark session create`, `az synapse pipeline create-run`).
- **MicroBurst**: workspace and identity enumeration.

## References

- [Microsoft: Synapse managed identity](https://learn.microsoft.com/azure/synapse-analytics/security/synapse-workspace-managed-identity)
- [HackTricks Cloud: Azure Synapse](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps: Azure managed identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
