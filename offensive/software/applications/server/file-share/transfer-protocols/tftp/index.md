---
title: "TFTP: attacking the Trivial File Transfer Protocol"
description: "TFTP is a minimal UDP file-transfer protocol on port 69 with no authentication whatsoever. Any client can read and, where the server allows, write files. The offensive surface is reading sensitive files the server exposes (configs, firmware, PXE boot files), writing to influence a device that consumes them, and directory traversal to escape the TFTP root."
keywords:
  - tftp
  - port 69
  - udp
  - no authentication
  - pxe
---

# TFTP

TFTP (Trivial File Transfer Protocol) is a deliberately minimal file-transfer protocol over UDP port 69. It has no authentication, no directory listing, and only two real operations: read a file (RRQ) and write a file (WRQ). It is used where simplicity matters, PXE network boot, router and phone firmware and configuration, backup of device configs, so the files it serves are often sensitive, and a writable TFTP server lets an attacker influence whatever device consumes those files. Because there is no auth, reaching port 69 is the only precondition.

```bash
# connect (no listing exists; you request known/guessed filenames)
tftp <target>
#   tftp> get <filename>      # read
#   tftp> put <localfile> <remote>   # write (if allowed)
# scripted
curl -s tftp://<target>/<filename> -o out
nmap -sU -p69 --script tftp-enum <target>     # probe for common filenames
```

## Subtopics

- **[File read and write](file-read-and-write.md)**: reading sensitive files and writing to the server.
- **[Directory traversal](directory-traversal.md)**: escaping the TFTP root with path traversal.

## References

- [RFC 1350 (TFTP)](https://datatracker.ietf.org/doc/html/rfc1350)
- [HackTricks: TFTP (69)](https://book.hacktricks.xyz/network-services-pentesting/69-udp-tftp)
