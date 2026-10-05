---
title: "SFTP: attacking SSH file transfer"
description: "Attacking SFTP, the SSH File Transfer Protocol: credential brute force and username enumeration against the SSH service it rides on, weak and stolen SSH keys, and escaping a restricted SFTP-only account out of its chroot or forced command to a full shell."
keywords:
  - SFTP
  - SSH
  - chroot escape
  - SSH keys
  - internal-sftp
---

# SFTP

SFTP runs over SSH, so it shares SSH's authentication surface: credential brute force, username enumeration, and key-based access. Its file-share-specific attack is escaping a restricted SFTP-only account, one confined by an OpenSSH chroot or a forced command, out to a real shell or the host filesystem. The broader SSH attack surface is covered under Remote Access.

## Subtopics

- **[Credential brute force](credential-brute-force.md)**: guessing SFTP logins.
- **[Username enumeration](username-enumeration.md)**: discovering valid users.
- **[Weak and stolen SSH keys](weak-and-stolen-ssh-keys.md)**: key-based access.
- **[Restricted shell and chroot escape](restricted-shell-and-chroot-escape.md)**: breaking out of an SFTP jail.

## References

- [HackTricks: pentesting SSH](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ssh.html)
- [OpenSSH sftp-server](https://man.openbsd.org/sftp-server.8)
