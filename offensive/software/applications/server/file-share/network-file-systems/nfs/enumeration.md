---
title: "Enumeration: discovering NFS exports and access"
description: "Enumerating an NFS server through the portmapper (rpcbind) and mount service to list exported directories, the hosts allowed to mount them, and their options, then mounting world-accessible exports to read their contents."
keywords:
  - NFS enumeration
  - showmount
  - rpcbind
  - portmapper
  - exports
---

# Enumeration

NFS registers with the portmapper (`rpcbind`, port 111), which points to the mount and NFS services. `showmount` lists the exported directories and the hosts allowed to mount each, and mounting a world-accessible export reveals its files and permissions.

```bash
rpcinfo -p <target>                         # RPC services (mountd, nfs)
showmount -e <target>                        # exported directories and allowed hosts
nmap -p 111,2049 --script nfs-ls,nfs-showmount,nfs-statfs <target>
mount -t nfs -o vers=3 <target>:/export /mnt/nfs    # mount and read
```

## Exploitation notes

- `showmount -e` is the fastest map; exports allowed to `*` or a broad subnet are mountable by anyone who can reach the server.
- The options behind each export (`no_root_squash`, `rw`, `insecure`) determine the next step; `nfs-ls` reads files without a full mount.
- A readable export is immediate loot; a `no_root_squash` or writable one is a write primitive.

## References

- [man 8 showmount](https://man7.org/linux/man-pages/man8/showmount.8.html)
- [HackTricks: pentesting NFS](https://book.hacktricks.wiki/en/network-services-pentesting/nfs-service-pentesting.html)
