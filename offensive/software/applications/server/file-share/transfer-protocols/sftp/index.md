---
title: "SFTP: attacking SSH-based file transfer"
description: "SFTP is a file-transfer subsystem of SSH on port 22, so it inherits SSH's authentication: passwords and public keys. The offensive surface is username enumeration, password brute force and spraying, weak or stolen SSH keys, and escaping the restricted shell or chroot that SFTP-only accounts are confined to, which turns transfer access into command execution."
keywords:
  - sftp
  - ssh
  - port 22
  - restricted shell
  - chroot
---

# SFTP

SFTP is not FTP over SSH but a distinct file-transfer subsystem built into the SSH server, reached on port 22 through the SSH connection. It therefore inherits SSH's security model entirely: authentication by password or public key, and the server's account and subsystem configuration. The offensive surface follows: enumerating valid usernames, brute-forcing or spraying passwords, abusing weak or stolen SSH keys, and, because SFTP-only accounts are typically confined to an `internal-sftp` restricted shell or a chroot, escaping that confinement into full command execution.

```bash
# SFTP rides SSH; enumerate and connect
nmap -p22 --script ssh2-enum-algos,ssh-auth-methods <target>
sftp user@<target>                            # subsystem access
ssh user@<target>                             # does the account also get a shell?
```

## Subtopics

- **[Username enumeration](username-enumeration.md)**: discovering valid accounts.
- **[Credential brute force](credential-brute-force.md)**: password attacks over SSH.
- **[Weak and stolen SSH keys](weak-and-stolen-ssh-keys.md)**: key-based access abuse.
- **[Restricted shell and chroot escape](restricted-shell-and-chroot-escape.md)**: breaking out of SFTP-only confinement.

## References

- [OpenSSH sftp-server and internal-sftp](https://man.openbsd.org/sftp-server.8)
- [RFC draft: SSH File Transfer Protocol](https://datatracker.ietf.org/doc/html/draft-ietf-secsh-filexfer)
- [HackTricks: SSH (22)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
