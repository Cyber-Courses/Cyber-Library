---
title: "FTP: attacking the File Transfer Protocol"
description: "Attacking FTP: anonymous access that exposes files without credentials, credential brute force against the cleartext login, and the FTP bounce trick that uses the PORT command to proxy connections and scan hosts the attacker cannot reach directly."
keywords:
  - FTP
  - anonymous FTP
  - brute force
  - FTP bounce
  - port 21
---

# FTP

FTP serves files over a cleartext control channel on port 21, with a separate data channel. It is attacked through anonymous access (a common default), brute force against its unencrypted login, and the FTP bounce quirk that abuses the PORT command to make the server open connections on the attacker's behalf.

## Subtopics

- **[Anonymous access](anonymous-access.md)**: reading files with no credentials.
- **[Credential brute force](credential-brute-force.md)**: guessing the cleartext login.
- **[FTP bounce scan](ftp-bounce-scan.md)**: proxying connections through the server.

## References

- [HackTricks: pentesting FTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ftp/index.html)
- [RFC 959: FTP](https://www.rfc-editor.org/rfc/rfc959)
