---
title: "TFTP: attacking the Trivial File Transfer Protocol"
description: "Attacking TFTP, the trivial, unauthenticated UDP file-transfer protocol used by network devices and PXE boot: reading and writing files with no authentication, and directory traversal where the server fails to confine requests to its root."
keywords:
  - TFTP
  - UDP 69
  - PXE
  - unauthenticated
  - directory traversal
---

# TFTP

TFTP is a minimal file-transfer protocol over UDP port 69 with no authentication at all, used for network-device configs, firmware, and PXE boot. Attacks are simple by nature: read and write files directly, and traverse outside the TFTP root where the server does not confine requests. It leaks device configurations that frequently contain credentials.

## Subtopics

- **[File read and write](file-read-and-write.md)**: unauthenticated GET and PUT.
- **[Directory traversal](directory-traversal.md)**: escaping the TFTP root.

## References

- [HackTricks: pentesting TFTP](https://book.hacktricks.wiki/en/network-services-pentesting/69-udp-tftp.html)
- [RFC 1350: TFTP](https://www.rfc-editor.org/rfc/rfc1350)
