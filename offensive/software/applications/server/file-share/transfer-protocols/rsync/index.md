---
title: "rsync: attacking the rsync daemon"
description: "rsync runs as a daemon on TCP 873 exposing named modules (shares), frequently with anonymous access and no authentication. The offensive surface is enumerating the modules, reading their contents anonymously, and abusing writable modules to plant files, with the daemon's privileges and any configured chroot shaping how far a write reaches."
keywords:
  - rsync
  - port 873
  - module
  - anonymous
  - rsync daemon
---

# rsync

rsync is best known as a file-sync tool over SSH, but it also runs as a standalone daemon listening on TCP 873, exposing named modules that behave like shares. Daemon modules are configured in `rsyncd.conf`, and they are commonly left anonymous (no `auth users`) and sometimes writable, with varying chroot and privilege settings. The offensive surface is enumerating the exposed modules, reading their contents without credentials, and writing to any module that allows it, which plants files on the host within whatever the daemon's chroot and privileges permit.

```bash
# list modules exposed by the rsync daemon (no credentials needed to list)
rsync rsync://<target>/                       # or: rsync <target>::
nmap -p873 --script rsync-list-modules <target>
```

## Subtopics

- **[Module enumeration](module-enumeration.md)**: listing modules and their access.
- **[Anonymous file access](anonymous-file-access.md)**: reading module contents without auth.
- **[Write access](write-access.md)**: planting files through a writable module.

## References

- [rsyncd.conf manual](https://download.samba.org/pub/rsync/rsyncd.conf.5)
- [rsync manual](https://download.samba.org/pub/rsync/rsync.1)
- [HackTricks: rsync (873)](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)
