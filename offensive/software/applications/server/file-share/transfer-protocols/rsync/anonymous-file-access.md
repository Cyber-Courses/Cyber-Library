---
title: "Anonymous file access: reading rsync module contents without credentials"
description: "rsync daemon modules without an auth users directive are readable by anyone, so an attacker downloads their entire contents anonymously. Because rsync modules frequently back up or publish server directories, those contents hold configurations, source, keys, and whole backups, making an anonymous module a direct data-theft foothold."
keywords:
  - rsync anonymous
  - download
  - backup
  - loot
  - rsync module
---

# Anonymous file access

An rsync module configured without `auth users` accepts any client, so its contents are fully readable anonymously. rsync modules are commonly used to back up or publish server directories, so what they expose is substantial: configuration trees, web and application source, SSH and TLS keys, and complete backup sets. Downloading an anonymous module is therefore a direct data-theft foothold, equivalent to reading a share, and rsync makes mirroring the whole tree trivial.

```bash
# mirror an entire anonymous module for offline loot
rsync -av rsync://<target>/<module>/ ./loot/
# or the double-colon form
rsync -av <target>::<module>/ ./loot/
# then mine the contents for secrets
grep -rinE 'password|secret|BEGIN (RSA|OPENSSH) PRIVATE KEY|api[_-]?key' ./loot | head
find ./loot -iname 'id_rsa' -o -iname '*.pem' -o -iname '*.conf' -o -iname '*.bak'
```

## Exploitation notes

- `rsync -av` mirrors the module recursively with attributes, pulling the whole tree efficiently for offline analysis; triage with the same credential and sensitive-file patterns used for any share loot.
- Backup modules are the richest: a module that holds server or database backups may yield whole systems (and, from a DC backup, domain hashes), as with backup-share looting.
- Read access is low-noise and needs no credentials; it is the obvious first action against any anonymous module found in [enumeration](module-enumeration.md).
- Recovered keys and configs pivot onward (SSH, services); feed them back into the environment.

## References

- [rsync manual](https://download.samba.org/pub/rsync/rsync.1)
- [HackTricks: rsync anonymous](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)
