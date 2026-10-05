---
title: "Transfer protocols: attacking file-transfer services"
description: "Attacking file-transfer services: anonymous and brute-forced access, bounce and relay tricks, cleartext exposure, and weak transport security across FTP, FTPS, SFTP, TFTP, and rsync."
keywords:
  - file transfer
  - FTP
  - SFTP
  - TFTP
  - rsync
---

# Transfer protocols

File-transfer services move files over a dedicated protocol rather than a mounted filesystem. They are attacked through weak or anonymous access, credential brute force, protocol quirks (FTP bounce), cleartext exposure, and daemon misconfiguration. They range from the cleartext legacy (FTP, TFTP) to the SSH-based (SFTP) and the daemon-based (rsync).

## Subtopics

- **[FTP](ftp/index.md)**: the cleartext file-transfer protocol.
- **[FTPS](ftps/index.md)**: FTP over TLS and its weak configurations.
- **[SFTP](sftp/index.md)**: SSH-based file transfer.
- **[TFTP](tftp/index.md)**: the trivial, unauthenticated UDP protocol.
- **[rsync](rsync/index.md)**: the rsync daemon and its modules.

## References

- [HackTricks: pentesting FTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ftp/index.html)
- [HackTricks: network services pentesting](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
