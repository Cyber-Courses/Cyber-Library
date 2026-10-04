---
title: "Cosmos DB: primary keys, resource tokens, and RBAC document access"
description: "Abusing Azure Cosmos DB: primary and read-only keys, resource tokens, and RBAC for document access."
keywords:
  - Cosmos DB
  - primary key
  - resource token
  - RBAC
  - documents
---

# Cosmos DB

Cosmos DB account access is key-based by default: the account's **primary key** grants full read and write to every database and container, and is handed out by a control-plane call to anyone with `Microsoft.DocumentDB/databaseAccounts/listKeys/action`. There are also read-only keys, scoped resource tokens, and a data-plane RBAC model, but the primary key is the prize because it bypasses all of them.

## Listing the keys

```bash
az cosmosdb keys list --name <acct> -g <rg> --type keys
az cosmosdb keys list --name <acct> -g <rg> --type read-only-keys
# connection strings (key embedded)
az cosmosdb keys list --name <acct> -g <rg> --type connection-strings
```

## Reading documents

```bash
# with the key, query through the SQL (core) API endpoint
az cosmosdb sql container query --account-name <acct> -g <rg> \
  -d <db> -c <container> --query-text "SELECT * FROM c"
```

## Exploitation notes

- `listKeys` is a control-plane action, so a principal with Contributor or a custom role carrying it gets full data access without any data-plane role assignment.
- Keys are static and rarely rotated, so a recovered primary key is durable access; it also appears in app settings and Automation assets, so check [app settings](../credentials/app-settings-and-connection-strings.md) first.
- Regenerate-and-steal is possible but noisy and breaks the application; prefer reading the existing key.

## Tools

- **az cli** (`az cosmosdb keys list`, `az cosmosdb sql container query`).
- **MicroBurst**: surfaces Cosmos DB keys during subscription enumeration.

## References

- [Microsoft: Cosmos DB secure access to data](https://learn.microsoft.com/azure/cosmos-db/secure-access-to-data)
- [HackTricks Cloud: Azure Cosmos DB](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
