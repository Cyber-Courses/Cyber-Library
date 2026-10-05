---
title: "UID and GID spoofing: impersonating users over NFS"
description: "Abusing the AUTH_SYS trust model of NFS, where the server trusts the numeric UID and GID the client sends, by setting a local user to the target's UID and GID to read and write that user's files on the export as if you were them."
keywords:
  - UID spoofing
  - GID spoofing
  - AUTH_SYS
  - NFS impersonation
  - file access
---

# UID and GID spoofing

With AUTH_SYS (the default for NFSv3), the server performs no real authentication: it trusts the UID and GID numbers the client puts on each request. A client with root (to create users or set IDs) simply becomes any user by matching their UID, then reads and writes that user's files on the export with their permissions.

```bash
mount -t nfs <target>:/export /mnt/nfs
ls -ln /mnt/nfs                              # see which UIDs own the files
# Create/switch to a local user with the target UID, then access as them
useradd -u 1005 victim && su victim -c 'cat /mnt/nfs/home/victim/.ssh/id_rsa'
```

## Exploitation notes

- This needs local root on the client (to set arbitrary UIDs), but no credential on the NFS server at all.
- Target UIDs that own interesting files: a developer's home directory, a service account's keys, root (which is where `no_root_squash` matters).
- NFSv4 with Kerberos (`sec=krb5`) defeats this, so check the export's security flavor; see [NFSv4 and Kerberos](nfsv4-and-kerberos.md).

## References

- [man 5 nfs](https://man7.org/linux/man-pages/man5/nfs.5.html)
- [HackTricks: pentesting NFS](https://book.hacktricks.wiki/en/network-services-pentesting/nfs-service-pentesting.html)
