---
title: "Azure data"
order: 6
description: "Attacking Azure data services: SQL Database access, Cosmos DB keys, Data Factory pipelines, Synapse Analytics, and storage tables."
keywords:
  - Azure data
  - SQL Database
  - Cosmos DB
  - Data Factory
  - Synapse
---

# Data

Azure's managed data services are reached two ways: by the service's own keys or connection strings (Cosmos DB keys, storage account keys), or by a principal whose Azure RBAC and database identity let it connect. The analytics services are different: Data Factory and Synapse run pipelines as their own **managed identity**, so compromising one is a path to that identity's token, not just to the data.

## Pages

- **[SQL Database](sql-database/index.md)**: Entra and SQL authentication, firewall reach, and the server managed identity.
- **[Cosmos DB](cosmos-db.md)**: primary and read-only keys, resource tokens, and RBAC document access.
- **[Data Factory](data-factory.md)**: pipelines and linked services that run as the factory managed identity.
- **[Synapse Analytics](synapse-analytics.md)**: SQL and Spark pools, pipelines, and the workspace managed identity.
- **[Storage Tables](storage-tables.md)**: Table storage read through account keys or SAS.

## References

- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Azure data services documentation](https://learn.microsoft.com/azure/)
