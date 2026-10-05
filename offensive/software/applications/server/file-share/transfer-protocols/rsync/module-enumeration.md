---
title: "Module enumeration: listing rsync daemon modules"
description: "Enumerating the modules an rsync daemon exports by querying it without a path, which returns the list of named directory trees it serves, revealing backup, mirror, and file shares to read from or write to."
keywords:
  - rsync module
  - module enumeration
  - rsync list
  - rsyncd.conf
  - port 873
---

# Module enumeration

An rsync daemon advertises its modules to anyone who connects without specifying a path. Listing them reveals the exported directory trees, their names often hinting at backups, web roots, or home directories, which guides whether to read or attempt to write.

```bash
# List modules (no path = list request)
rsync rsync://<target>/
rsync -av --list-only rsync://<target>/
nmap -p 873 --script rsync-list-modules <target>
```

## Exploitation notes

- Module names are descriptive (`backup`, `www`, `home`), pointing straight at high-value trees.
- A module that lists without credentials is readable anonymously; one that prompts for auth needs credentials from `rsyncd.secrets` or brute force.
- Note each module's apparent intent to decide read ([Anonymous file access](anonymous-file-access.md)) versus write ([Write access](write-access.md)).

## References

- [Nmap rsync-list-modules](https://nmap.org/nsedoc/scripts/rsync-list-modules.html)
- [HackTricks: pentesting rsync](https://book.hacktricks.wiki/en/network-services-pentesting/873-pentesting-rsync.html)
