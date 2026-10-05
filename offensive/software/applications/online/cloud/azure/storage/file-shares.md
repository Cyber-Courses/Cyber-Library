---
title: "File shares: reaching Azure Files over SMB"
description: "Reaching Azure Files SMB shares through storage keys or identity-based access for stored data."
keywords:
  - Azure Files
  - SMB
  - file share
  - storage key
  - data
---

# File shares

Azure Files exposes SMB (and NFS) shares backed by a storage account, mounted by servers and workstations for home directories, application data, and backups. Access is by the **storage account key** (which authenticates straight to the share) or by **identity-based** auth (Entra Kerberos or on-prem AD DS) mapped to RBAC. A recovered account key is the fastest path: it mounts any share on the account.

## Listing and reading shares

```bash
# enumerate shares and files with the account key
az storage share list --account-name acme --account-key <KEY>
az storage file download --account-name acme --account-key <KEY> \
  --share-name profiles --path 'user/secrets.config' --dest ./secrets.config
```

## Mounting over SMB

```bash
# mount the share directly with the account key as the password
mount -t cifs //acme.file.core.windows.net/profiles /mnt/share \
  -o vers=3.0,username=acme,password=<KEY>,dir_mode=0777,file_mode=0777
```

## Exploitation notes

- The account key doubles as the SMB password, so `listkeys` on the storage account is full share access without any file-share-specific permission.
- Azure Files is reachable over the internet on port 445 when the account has no network restriction, so a key plus an open account is remote data access with no foothold in the VNet.
- Look for mounted shares referenced in VM `fstab`, scripts, and scheduled tasks to find which shares hold the valuable data.

## Tools

- **az CLI** (`storage share`, `storage file`): list and pull files.
- **cifs-utils** (`mount -t cifs`): mount the share with the key.
- **MicroBurst**: recovers the account keys that unlock the shares.

## References

- [HackTricks Cloud: Azure storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [Microsoft: mount Azure Files on Linux](https://learn.microsoft.com/azure/storage/files/storage-how-to-use-files-linux)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
