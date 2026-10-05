---
title: "Directory traversal: escaping the TFTP root"
description: "Reading and writing files outside the TFTP server's base directory with path-traversal sequences, where the server fails to confine filenames to its root, exposing system files such as password and configuration files beyond the intended TFTP directory."
keywords:
  - TFTP traversal
  - path traversal
  - directory escape
  - file disclosure
  - ../
---

# Directory traversal

A correctly implemented TFTP server confines requests to its root directory. Where it does not, filenames containing traversal sequences reach files elsewhere on the host, turning a limited file server into arbitrary file read (and, with write enabled, write) of system files outside the TFTP directory.

```bash
tftp <target>
tftp> get ../../../../etc/passwd loot_passwd
tftp> get ..\..\..\..\windows\win.ini loot_winini   # Windows TFTP servers
```

## Exploitation notes

- Try both `../` and `..\` separators depending on the server's platform, and repeat the sequence well past the expected depth.
- Target `/etc/passwd` and service configs on Unix, and known config and credential files on Windows TFTP implementations.
- Where write is also unconfined, traversal plus PUT is an arbitrary file write, reaching cron, startup, or web paths.

## References

- [HackTricks: pentesting TFTP](https://book.hacktricks.wiki/en/network-services-pentesting/69-udp-tftp.html)
- [RFC 1350: TFTP](https://www.rfc-editor.org/rfc/rfc1350)
