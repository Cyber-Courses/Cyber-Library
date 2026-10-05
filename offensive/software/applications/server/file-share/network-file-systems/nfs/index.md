---
title: "NFS: attacking the Unix network file system"
description: "NFS exports Unix filesystems over the network, and with the traditional AUTH_SYS security it trusts the client-supplied user and group IDs entirely. The offensive surface is enumerating exports and their options, abusing no_root_squash to write files as root, spoofing UIDs and GIDs to read and write any user's files, and the weaker points of NFSv4 with Kerberos."
keywords:
  - nfs
  - auth_sys
  - no_root_squash
  - exports
  - showmount
---

# NFS

NFS (Network File System) exports Unix directories over the network. Its defining weakness is the traditional `AUTH_SYS` (`sec=sys`) security model, under which the server trusts whatever user and group IDs the client includes in each request: there is no authentication of the client's claimed identity, only the IDs it asserts. That makes an NFS export with default options a near-open door. The surface is enumerating the exports and their options, abusing `no_root_squash` to act as root on the export, spoofing arbitrary UIDs/GIDs to access any user's files, and the comparatively harder NFSv4-with-Kerberos configurations.

```bash
# discover exports and the RPC services behind NFS
showmount -e <target>                         # exported paths and allowed clients
rpcinfo -p <target>                           # mountd/nfs/portmapper ports
nmap -p111,2049 --script nfs-showmount,nfs-ls <target>
```

## Subtopics

- **[Enumeration](enumeration.md)**: exports, options, and mounting.
- **[no_root_squash abuse](no-root-squash-abuse.md)**: writing files as root on an export.
- **[UID and GID spoofing](uid-and-gid-spoofing.md)**: impersonating any user under AUTH_SYS.
- **[NFSv4 and Kerberos](nfsv4-and-kerberos.md)**: the stronger configuration and its weak points.

## References

- [Linux exports(5) and NFS security](https://man7.org/linux/man-pages/man5/exports.5.html)
- [RFC 1813 (NFSv3) and RFC 8881 (NFSv4)](https://datatracker.ietf.org/doc/html/rfc8881)
- [HackTricks: NFS (2049)](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)
