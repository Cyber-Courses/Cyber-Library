---
title: "Azure storage"
order: 4
description: "Attacking Azure storage: blob container enumeration and access, SAS token abuse, disk snapshot theft, and Azure Files shares."
keywords:
  - Azure storage
  - blob
  - SAS token
  - disk snapshot
  - file share
---

# Storage

Storage is where the data lives, so it is the usual objective once a foothold or a storage key is held. Azure storage breaks into object storage (**Blob**, exposed through public containers, account keys, and SAS), block storage (managed **disk snapshots** restored offline), and shared file systems (**Azure Files** over SMB). Most compromise here is exposure or key theft rather than an exploit: an anonymous container, a leaked SAS URL with a year of validity, a snapshot exported to a disk you control.

## What folds in here

- **[Blob storage](blob-storage/index.md)**: finding public containers and reading blobs through account keys, RBAC, or anonymous access.
- **[SAS tokens](sas-tokens.md)**: over-scoped account, service, and user-delegation shared access signatures.
- **[Disk snapshots](disk-snapshots.md)**: snapshotting and exporting a managed disk to read a VM's OS disk offline.
- **[File shares](file-shares.md)**: Azure Files SMB shares reached through storage keys or identity-based access.

Enumeration folds into each page: finding the account or share is the first half of reaching its data. The account key, where recovered, is the master credential for every data plane below and is covered under [credentials](../credentials/index.md).

## References

- [HackTricks Cloud: Azure Blob storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: authorizing access to blob data](https://learn.microsoft.com/azure/storage/common/authorize-data-access)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
