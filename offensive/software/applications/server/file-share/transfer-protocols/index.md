---
title: "Transfer protocols: attacking file-transfer services"
description: "File-transfer services move files over dedicated protocols: FTP and its TLS variant FTPS, SFTP over SSH, the trivial TFTP, and rsync. Each has its own weaknesses, anonymous and default access, credential brute force, directory traversal, weak transport security, and restricted-shell escapes, that turn a transfer endpoint into data access or a foothold."
keywords:
  - ftp
  - sftp
  - tftp
  - rsync
  - file transfer
---

# Transfer protocols

Beyond mountable network file systems, files move over dedicated transfer services, and each is its own attack surface. FTP is old and plaintext, with anonymous access and bounce quirks; FTPS wraps it in TLS that is often misconfigured; SFTP rides SSH and inherits its credential and key weaknesses plus restricted-shell escapes; TFTP is a tiny, authentication-free UDP protocol prone to traversal; and rsync daemons frequently expose modules anonymously. The recurring moves are anonymous or default access, credential attacks, directory traversal, weak transport security, and escaping a restricted transfer-only shell into a real one.

```bash
# identify transfer services on a host
nmap -p21,22,69,873,990 -sV <target>        # FTP, SSH/SFTP, TFTP, rsync, FTPS
nmap -p21 --script ftp-anon,ftp-syst <target>
```

## Subtopics

- **[FTP](ftp/index.md)**: anonymous access, credential brute force, and the bounce quirk.
- **[FTPS](ftps/index.md)**: FTP over TLS and its certificate and configuration weaknesses.
- **[SFTP](sftp/index.md)**: SSH-based transfer, keys, and restricted-shell escape.
- **[TFTP](tftp/index.md)**: the authentication-free trivial protocol and traversal.
- **[rsync](rsync/index.md)**: rsync daemon modules and anonymous access.

## References

- [RFC 959 (FTP)](https://datatracker.ietf.org/doc/html/rfc959)
- [RFC 913 / RFC 1350 (TFTP)](https://datatracker.ietf.org/doc/html/rfc1350)
- [rsync documentation](https://rsync.samba.org/documentation.html)
