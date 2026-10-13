---
title: "Cosmos DB : clés primaires, resource tokens et accès aux documents par RBAC"
order: 2
description: "Abuser d'Azure Cosmos DB : clés primaires et en lecture seule, resource tokens et RBAC pour l'accès aux documents."
keywords:
  - Cosmos DB
  - clé primaire
  - resource token
  - RBAC
  - documents
---

# Cosmos DB

L'accès à un compte Cosmos DB est basé sur les clés par défaut : la **clé primaire** du compte accorde un accès complet en lecture et en écriture à toutes les bases de données et à tous les conteneurs, et est distribuée par un appel du plan de contrôle à quiconque détient `Microsoft.DocumentDB/databaseAccounts/listKeys/action`. Il existe aussi des clés en lecture seule, des resource tokens limités et un modèle RBAC de plan de données, mais la clé primaire est le butin car elle les contourne tous.

## Lister les clés

```bash
az cosmosdb keys list --name <acct> -g <rg> --type keys
az cosmosdb keys list --name <acct> -g <rg> --type read-only-keys
# connection strings (key embedded)
az cosmosdb keys list --name <acct> -g <rg> --type connection-strings
```

## Lire les documents

```bash
# with the key, query through the SQL (core) API endpoint
az cosmosdb sql container query --account-name <acct> -g <rg> \
  -d <db> -c <container> --query-text "SELECT * FROM c"
```

## Notes d'exploitation

- `listKeys` est une action de plan de contrôle, de sorte qu'un principal doté de Contributor ou d'un rôle personnalisé qui la porte obtient un accès complet aux données sans aucune attribution de rôle de plan de données.
- Les clés sont statiques et rarement renouvelées, de sorte qu'une clé primaire récupérée est un accès durable ; elle apparaît aussi dans les app settings et les assets Automation, donc vérifiez d'abord les [app settings](../credentials/app-settings-and-connection-strings.md).
- Régénérer-puis-voler est possible mais bruyant et casse l'application ; préférez lire la clé existante.

## Outils

- **az cli** (`az cosmosdb keys list`, `az cosmosdb sql container query`).
- **MicroBurst** : fait remonter les clés Cosmos DB lors de l'énumération de l'abonnement.

## Références

- [Microsoft : accès sécurisé aux données Cosmos DB](https://learn.microsoft.com/azure/cosmos-db/secure-access-to-data)
- [HackTricks Cloud : Azure Cosmos DB](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
