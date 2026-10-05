---
title: "File Share: attacking file-sharing and transfer services"
description: "Attacking file-sharing and transfer services: network file systems (SMB, NFS, AFP) reached by mounting or protocol clients, transfer protocols (FTP, FTPS, SFTP, TFTP, rsync), and web-based file access (HTTP file servers, WebDAV, managed file transfer appliances). Covers anonymous and weak access, writable-share abuse, and loot."
keywords:
  - file sharing
  - SMB
  - NFS
  - FTP
  - managed file transfer
---

# File Share

File shares are where an environment keeps its data, and they are routinely exposed with weak or no authentication, world-readable or world-writable permissions, and legacy protocols. Offensive value is in two directions: reading what the share holds (credentials, backups, documents) and writing to it to gain execution or coerce other users. The area splits by how the files are reached.

## Subtopics

- **[Network file systems](network-file-systems/index.md)**: SMB, NFS, and AFP, reached by mounting or protocol clients.
- **[Transfer protocols](transfer-protocols/index.md)**: FTP, FTPS, SFTP, TFTP, and rsync.
- **[Web file access](web-file-access/index.md)**: HTTP file servers, WebDAV, and managed file transfer appliances.

## References

- [HackTricks: pentesting network services](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
