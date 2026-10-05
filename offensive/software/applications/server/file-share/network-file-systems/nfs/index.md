---
title: "NFS: attacking Unix network file system exports"
description: "Attacking NFS exports: enumerating them through the portmapper and mount service, abusing the no_root_squash option to write files as root on the server, spoofing UID and GID to impersonate users under AUTH_SYS, and the NFSv4 and Kerberos specifics."
keywords:
  - NFS
  - exports
  - no_root_squash
  - AUTH_SYS
  - showmount
---

# NFS

NFS exports server directories to clients that mount them. Its classic weakness is the trust model: with AUTH_SYS, the server trusts the UID and GID the client sends, so a client that controls its own UIDs can impersonate any user, and the `no_root_squash` export option lets a client's root be root on the server's files. Enumeration through the portmapper reveals what is exported and to whom.

## Subtopics

- **[Enumeration](enumeration.md)**: discovering exports and their access.
- **[no_root_squash abuse](no-root-squash-abuse.md)**: writing files as root on the server.
- **[UID and GID spoofing](uid-and-gid-spoofing.md)**: impersonating users under AUTH_SYS.
- **[NFSv4 and Kerberos](nfsv4-and-kerberos.md)**: the NFSv4 model and sec=krb5.

## References

- [HackTricks: pentesting NFS](https://book.hacktricks.wiki/en/network-services-pentesting/nfs-service-pentesting.html)
- [man 5 exports](https://man7.org/linux/man-pages/man5/exports.5.html)
