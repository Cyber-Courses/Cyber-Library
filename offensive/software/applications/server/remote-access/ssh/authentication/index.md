---
title: "Authentication: attacking SSH login"
order: 1
description: "SSH authenticates by password or public key, and both are attack surfaces: default and weak passwords yield to brute force and spraying, exposed or reused private keys authenticate with no password, and keys from broken generators are predictable. The authentication method the server offers per user also shapes which attack applies."
keywords:
  - ssh authentication
  - password
  - public key
  - default credentials
  - weak keys
---

# Authentication

SSH grants access by password or by public-key authentication, and the server's configuration decides which is offered. Both are attacked. Passwords fall to default-credential checks, brute force, and spraying, bounded by any rate limiting. Public-key authentication, the more common real mechanism, is attacked not by breaking the key but by finding the private key, exposed in shares, backups, and repositories, or reused across hosts, and by exploiting keys generated weakly enough to be predictable. Enumerating which method the server accepts per user directs the effort.

```bash
# which authentication methods does the server offer for a user?
ssh -o PreferredAuthentications=none -o PubkeyAuthentication=no user@<target> 2>&1 | grep -i 'authentications that can continue'
```

## Subtopics

- **[Default credentials](default-credentials.md)**: vendor and factory SSH logins.
- **[Password brute force](password-brute-force.md)**: online password attacks and spraying.
- **[Public key exposure](public-key-exposure.md)**: finding and reusing private keys.
- **[Weak key generation](weak-key-generation.md)**: predictable keys from broken generators.

## References

- [OpenSSH sshd_config authentication options](https://man.openbsd.org/sshd_config)
- [HackTricks: SSH authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
