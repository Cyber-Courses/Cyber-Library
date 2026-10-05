---
title: "Backups and archives: looting bulk data from shares"
description: "Looting the backups and archives teams leave on SMB shares: virtual machine and database backups, VHD and VMDK disk images, mailbox exports, and password-database files, each of which contains credentials and data in bulk without needing access to the live system."
keywords:
  - backups
  - VHD VMDK
  - database backup
  - KeePass
  - bulk loot
---

# Backups and archives

Shares are where backups land, and a backup is a complete copy of a system's data without its live protections. Virtual machine and database backups, VHD and VMDK images, mailbox (`.pst`) exports, and password-database files (`.kdbx`) on a readable share hand over credentials and data in bulk, offline, with no need to touch the running system.

```bash
# Find backup and archive artifacts on shares
find /mnt/share -iregex '.*\.\(bak\|vhdx?\|vmdk\|pst\|kdbx\|7z\|zip\|sql\)$' 2>/dev/null
# A VHD/VMDK or system backup yields SAM/SYSTEM or NTDS.dit offline
guestmount -a disk.vhdx -i --ro /mnt/vhd
```

## Exploitation notes

- A domain controller backup or an NTDS.dit export is a full credential database; a member backup yields `SAM`/`SYSTEM` for local hashes.
- A `.kdbx` password database on a share is high-value even before cracking, since users name entries descriptively; crack offline.
- Database `.bak` and `.sql` dumps hold application credentials and customer data directly.

## References

- [libguestfs guestmount](https://libguestfs.org/guestmount.1.html)
- [HackTricks: pentesting SMB](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
