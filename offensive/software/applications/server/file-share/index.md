---
title: "File Share: attacking file-sharing and transfer services"
order: 4
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

## Find the file services

One sweep identifies the file services on a host and points to the right subtopic:

```bash
nmap -sV -p21,22,69,111,139,445,548,873,990,2049,80,443 <target>
#  445/139 SMB   2049/111 NFS   548 AFP   21 FTP   990 FTPS   22 SFTP(SSH)
#  69 TFTP(UDP)  873 rsync   80/443 HTTP file server / WebDAV / MFT portal
# quick access probes
nxc smb <target> -u '' -p '' --shares          # SMB null-session shares
showmount -e <target>                           # NFS exports
rsync <target>::                                # rsync modules
curl -s -X OPTIONS http://<target>/ -i | grep -i dav   # WebDAV
```

Read the open ports to the subtopic: 445/2049/548 to [network file systems](network-file-systems/index.md), 21/22/69/873 to [transfer protocols](transfer-protocols/index.md), and 80/443 serving files to [web file access](web-file-access/index.md). Then, for each, the recurring questions are the same: is access anonymous or weak, is it readable (loot) or writable (execution and coercion), and does the server implementation have a known flaw.

## Subtopics

- **[Network file systems](network-file-systems/index.md)**: SMB, NFS, and AFP, reached by mounting or protocol clients.
- **[Transfer protocols](transfer-protocols/index.md)**: FTP, FTPS, SFTP, TFTP, and rsync.
- **[Web file access](web-file-access/index.md)**: HTTP file servers, WebDAV, and managed file transfer appliances.

## References

- [HackTricks: pentesting network services](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
