---
title: "Azure Blob storage"
description: "Attacking Azure Blob storage: finding public containers and reading blobs through account keys, RBAC, or anonymous access."
keywords:
  - blob storage
  - container
  - anonymous access
  - account key
  - public blob
---

# Blob storage

Azure Blob storage holds objects in **containers** inside a **storage account**, addressed at `https://<account>.blob.core.windows.net/<container>`. Access comes three ways, and each is an attack path: **anonymous** when a container's public access level is set to blob or container, an **account key** (the master credential for the whole account), or **Azure RBAC** data roles such as Storage Blob Data Reader. The work is finding the account, finding a container, and reaching its blobs by whichever of the three is open.

## What folds in here

- **[Enumeration](enumeration.md)**: discovering storage accounts and public containers through naming guesses and anonymous listing.
- **[Access](access.md)**: reading and writing blobs through account keys, RBAC data roles, or misconfigured anonymous access.

## References

- [HackTricks Cloud: Azure storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [NetSPI: anonymous blob access and MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: blob public access levels](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure)
