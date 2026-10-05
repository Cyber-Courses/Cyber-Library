---
title: "Network file systems: attacking SMB, NFS, and AFP"
description: "Attacking network file systems reached by mounting or protocol clients: SMB (the dominant enterprise share protocol), NFS (Unix exports and their trust model), and AFP (Apple file sharing and Netatalk). Covers enumeration, weak authentication, writable-share abuse, and looting."
keywords:
  - network file system
  - SMB
  - NFS
  - AFP
  - mounted share
---

# Network file systems

Network file systems present remote storage as mountable shares. They are attacked through their enumeration and authentication (often null, guest, or trust-based), the permissions on the shares themselves, and the trust model each protocol uses to map identities. SMB dominates enterprise environments, NFS is the Unix standard, and AFP is the Apple legacy.

## Subtopics

- **[SMB](smb/index.md)**: the dominant enterprise share protocol.
- **[NFS](nfs/index.md)**: Unix exports and their UID trust model.
- **[AFP](afp/index.md)**: Apple Filing Protocol and Netatalk.

## References

- [HackTricks: pentesting SMB](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
- [HackTricks: pentesting NFS](https://book.hacktricks.wiki/en/network-services-pentesting/nfs-service-pentesting.html)
