---
title: "Access: reading blobs through keys, RBAC, or anonymous access"
description: "Reading and writing Azure blobs through account keys, RBAC data roles, or misconfigured anonymous access."
keywords:
  - blob access
  - account key
  - RBAC
  - anonymous
  - data plane
---

# Access

Once a container is located, reading its blobs depends on which authorization the account exposes. In descending order of attacker value: a recovered **account key** (full control of every container, every operation), an **Azure RBAC** data role on the account or container (Storage Blob Data Reader or Contributor), or **anonymous** access where the public level allows it.

## With an account key

The account key is the master data-plane credential. Recover it from the management plane (if you hold `Microsoft.Storage/storageAccounts/listkeys/action`) or from leaked config:

```bash
az storage account keys list --account-name acme -g rg --query '[0].value' -o tsv
# then read with the key
az storage blob list   --account-name acme --account-key <KEY> --container-name backups
az storage blob download --account-name acme --account-key <KEY> -c backups -n secrets.tar -f ./secrets.tar
```

## With an RBAC data role

A principal holding a Storage Blob Data role reads without the key, using its own token:

```bash
az storage blob download --account-name acme --auth-mode login \
  -c backups -n secrets.tar -f ./secrets.tar
```

Note that management-plane roles such as Contributor do not grant data-plane read by default, but they can read the account key (`listkeys`), which does, so Contributor over a storage account is effectively full data access.

## Anonymous

```bash
curl -O "https://acme.blob.core.windows.net/backups/secrets.tar"
```

## Exploitation notes

- `listkeys` is the quiet escalation: a management-plane role that can read keys bypasses the data-plane RBAC entirely.
- Look across all four services on the account (blob, file, queue, table) with the same key; operators often forget the key unlocks every one.
- Storage Explorer and `azcopy` with the key or a SAS pull whole containers fast for offline triage.

## Tools

- **az CLI** / **azcopy**: key, SAS, and `--auth-mode login` reads and bulk copy.
- **Azure Storage Explorer**: point-and-click browsing with a key or SAS.
- **MicroBurst** (`Get-AzStorageKeysREST`): pull account keys over REST from a token.

## References

- [HackTricks Cloud: Azure storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [Microsoft: authorize access to blobs](https://learn.microsoft.com/azure/storage/common/authorize-data-access)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
