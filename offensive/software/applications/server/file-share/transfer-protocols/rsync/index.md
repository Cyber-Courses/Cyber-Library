---
title: "rsync: attacking the rsync daemon"
description: "Attacking an rsync daemon on port 873: enumerating its modules (exported directory trees), reading files from anonymous or weakly-authenticated modules, and abusing writable modules to upload files to sensitive locations for code execution."
keywords:
  - rsync
  - rsyncd
  - module
  - port 873
  - anonymous
---

# rsync

The rsync daemon (`rsyncd`, port 873) exports directory trees as named modules. It is often deployed for backups and mirrors with weak or no authentication. Attacks enumerate the modules, read from anonymous or weakly-authenticated ones, and abuse writable modules to plant files that grant code execution.

## Subtopics

- **[Module enumeration](module-enumeration.md)**: listing the exported modules.
- **[Anonymous file access](anonymous-file-access.md)**: reading from open modules.
- **[Write access](write-access.md)**: uploading to writable modules for execution.

## References

- [HackTricks: pentesting rsync](https://book.hacktricks.wiki/en/network-services-pentesting/873-pentesting-rsync.html)
- [rsync manual](https://download.samba.org/pub/rsync/rsync.1)
