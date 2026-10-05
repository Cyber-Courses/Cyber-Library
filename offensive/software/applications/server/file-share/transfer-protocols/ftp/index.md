---
title: "FTP: attacking the File Transfer Protocol"
description: "FTP serves files on TCP 21 in cleartext, with a control channel and separate data connections. The offensive surface is anonymous access that many servers allow, credential brute force against its plaintext login, the FTP bounce feature that proxies port scans through the server, and credential capture from its unencrypted traffic."
keywords:
  - ftp
  - port 21
  - anonymous
  - ftp bounce
  - cleartext
---

# FTP

FTP (File Transfer Protocol) serves files on TCP 21 using a cleartext control channel and separate data connections (active or passive). Its age shows in its security: credentials and data travel unencrypted, anonymous access is a built-in feature frequently left enabled, and the `PORT` command enables the bounce technique that turns the server into a scan proxy. The surface is therefore anonymous access, brute force against the plaintext login, the bounce quirk, and passive capture of credentials and files from the unencrypted stream.

```bash
nmap -p21 --script ftp-anon,ftp-syst,ftp-bounce <target>
ftp <target>                                  # interactive; try anonymous first
```

## Subtopics

- **[Anonymous access](anonymous-access.md)**: the built-in anonymous login.
- **[Credential brute force](credential-brute-force.md)**: attacking the plaintext login.
- **[FTP bounce scan](ftp-bounce-scan.md)**: proxying port scans through the server.

## References

- [RFC 959 (FTP)](https://datatracker.ietf.org/doc/html/rfc959)
- [HackTricks: FTP (21)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp)
