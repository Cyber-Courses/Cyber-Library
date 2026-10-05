---
title: "Anonymous file access: reading from open rsync modules"
description: "Reading and downloading files from rsync modules that require no authentication, pulling backups, web roots, and home directories in bulk from a daemon that exposes them anonymously."
keywords:
  - rsync anonymous
  - rsync download
  - unauthenticated
  - backup access
  - rsync copy
---

# Anonymous file access

Modules configured without `auth users` are readable by anyone. A single `rsync` command pulls the entire module, so an anonymous backup or web-root module hands over its full contents, which commonly include credentials, source, and data.

```bash
# Copy an entire anonymous module locally
rsync -av rsync://<target>/backup/ ./loot/
# Pull a specific path
rsync -av rsync://<target>/home/user/.ssh/ ./keys/
```

## Exploitation notes

- Mirror the whole module first; backups and home directories hold keys, configs, and credentials.
- Even read-only access to a web-root module leaks source code and its embedded secrets.
- Preserve attributes (`-a`) to keep timestamps and permissions that may matter for later analysis.

## References

- [rsync manual](https://download.samba.org/pub/rsync/rsync.1)
- [HackTricks: pentesting rsync](https://book.hacktricks.wiki/en/network-services-pentesting/873-pentesting-rsync.html)
