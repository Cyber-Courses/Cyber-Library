---
title: "Storage Tables: reading Table storage with keys or SAS"
description: "Reading Azure Table storage through account keys or SAS for structured data."
keywords:
  - Storage Tables
  - Table storage
  - account key
  - SAS
  - NoSQL
---

# Storage Tables

Azure Table storage is the key-value NoSQL service inside a storage account, and it shares the account's auth: the **account key** or a **SAS** token reads and writes every table. Applications use it for session state, configuration, and audit data, so a recovered storage key often exposes tables that hold tokens, user records, or internal state.

## Listing and reading tables

```bash
KEY=$(az storage account keys list -n <acct> -g <rg> --query '[0].value' -o tsv)
az storage table list --account-name <acct> --account-key "$KEY" -o table
az storage entity query --account-name <acct> --account-key "$KEY" --table-name <t>
```

## Exploitation notes

- Table access rides on the same account key as [blob storage](../storage/blob-storage/index.md), so one `listKeys` grants both; check what the key already unlocks before anything louder.
- A SAS token scoped to the table service reaches tables even without the account key, and such tokens leak in app config and URLs.
- Tables backing application authorization (role rows, feature flags) make a write an application-level escalation.

## Tools

- **az cli** (`az storage table list`, `az storage entity query`).
- **Azure Storage Explorer**: browse tables with a key or SAS.

## References

- [Microsoft: Table storage authorization](https://learn.microsoft.com/azure/storage/tables/authorize-access-azure-active-directory)
- [HackTricks Cloud: Azure storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
