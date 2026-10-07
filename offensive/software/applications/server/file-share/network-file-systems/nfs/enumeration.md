---
title: "Enumeration: NFS exports, options, and mounting"
order: 1
description: "NFS enumeration lists the exported paths, the clients allowed to mount them, and, where readable, the export options that decide squashing and access. Mounting an export exposes its files under the client-asserted identity, and the file ownership seen after mounting reveals which UIDs and GIDs to spoof for fuller access."
keywords:
  - showmount
  - exports
  - mount
  - nfsstat
  - uid mapping
---

# Enumeration

Enumerating NFS answers what is exported, to whom, and under what options, and then mounting reveals the file ownership that drives the spoofing and squash attacks. The export list and allowed-client specification come from the server; the critical options (`root_squash` vs `no_root_squash`, `all_squash`, `sec=`) are not always visible remotely, so mounting and inspecting ownership is how you learn the effective model.

```bash
# list exports and permitted clients
showmount -e <target>                          # e.g. /srv/share *  (world) or 10.0.0.0/8
rpcinfo -p <target>                            # confirm mountd/nfs reachable
# mount an export and inspect
mkdir /mnt/nfs && mount -t nfs -o vers=3 <target>:/srv/share /mnt/nfs
ls -lan /mnt/nfs                               # numeric UID/GID ownership of files
mount -t nfs -o vers=4 <target>:/ /mnt/nfs     # NFSv4 presents a single pseudo-root
```

Read `ls -lan` numerically: the UID/GID that owns each file is what you must present to read or write it under `AUTH_SYS`. Files owned by UID 0 test whether root is squashed; files owned by other UIDs name the identities to spoof.

## Exploitation notes

- `showmount -e` with a wildcard or broad client spec means anyone who can reach 2049 can mount; a restricted client list may be bypassable by spoofing the source address on a flat network.
- The export options are the real security; since they are often not visible remotely, mount and test, create a file and see what UID it lands as, and read a root-owned file, to learn whether `root_squash`/`all_squash` apply.
- NFSv4 uses a single exported pseudo-filesystem and name-based id mapping rather than NFSv3's per-export numeric model; mount both versions to see what each exposes.
- The ownership map feeds [UID and GID spoofing](uid-and-gid-spoofing.md) and the [no_root_squash](no-root-squash-abuse.md) test.

## Tools

- [nfs-common (showmount, mount.nfs)](https://man7.org/linux/man-pages/man8/showmount.8.html)
- [nmap nfs scripts](https://nmap.org/nsedoc/)

## References

- [exports(5)](https://man7.org/linux/man-pages/man5/exports.5.html)
- [HackTricks: NFS](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)
