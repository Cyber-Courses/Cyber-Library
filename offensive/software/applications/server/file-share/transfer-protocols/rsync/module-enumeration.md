---
title: "Module enumeration: listing rsync daemon modules and access"
description: "The rsync daemon lists its modules to any client that asks, revealing the share names and their comments. Probing each module shows whether it requires authentication and whether it is readable or writable, producing the map that directs looting and file-planting against the daemon."
keywords:
  - rsync modules
  - rsyncd
  - enumeration
  - listing
  - access
---

# Module enumeration

The rsync daemon advertises its modules to anyone who connects, with no credentials required for the listing, so enumeration is immediate. Each module is a named export mapping to a server directory; the listing shows the names and comments, and probing each module reveals whether it demands authentication (`auth users`) and whether the current access is read or write. That map, which modules are anonymous, readable, and writable, directs everything that follows.

```bash
# list all modules and their comments
rsync rsync://<target>/
# equivalent double-colon syntax
rsync <target>::
# probe a specific module: does it need auth? is it listable?
rsync rsync://<target>/<module>/              # lists the module's top-level contents
rsync -av --list-only rsync://<target>/<module>/   # recursive-ish listing of entries
```

Read the results: a module that lists its contents without prompting for a password is anonymous-readable; one that errors with an auth requirement needs credentials; writability is confirmed separately by attempting an upload.

## Exploitation notes

- The module listing is always available to probe (it is how clients discover shares), so this is a reliable first step needing no access.
- Module names and comments reveal purpose (backups, web content, config, home directories), prioritising which to loot or target for writes.
- An anonymous module that lists contents is immediately readable, see [Anonymous file access](anonymous-file-access.md); test each for writes separately, see [Write access](write-access.md).
- Modules requiring `auth users` need credentials (rsync daemon auth uses a secrets file, not system accounts); note them for credential attacks but focus first on the anonymous ones.

## Tools

- [rsync](https://download.samba.org/pub/rsync/rsync.1)
- [nmap rsync-list-modules](https://nmap.org/nsedoc/scripts/rsync-list-modules.html)

## References

- [rsyncd.conf](https://download.samba.org/pub/rsync/rsyncd.conf.5)
- [HackTricks: rsync](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)
