---
title: "Authentication: attacking SSH login"
order: 1
description: "SSH can use public keys and certificates, passwords, keyboard-interactive, host-based authentication, GSSAPI, hardware-backed keys, or a required sequence of methods. Assess the effective policy for the specific user, source, and server configuration."
keywords:
  - ssh authentication
  - password
  - public key
  - default credentials
  - weak keys
---

# Authentication

OpenSSH can authenticate clients through public keys or certificates, passwords, keyboard-interactive mechanisms, host-based authentication, GSSAPI, and hardware-backed security keys. `AuthenticationMethods` can also require a sequence such as public key followed by keyboard-interactive. The effective methods may vary with `Match` rules, username, source address, groups, and other connection attributes. Review the complete authentication chain before assessing password guessing, exposed private keys, weak key generation, certificate trust, or multifactor controls.

```bash
# non-interactive diagnostic; inspect the server-reported continuation methods
ssh -vvv -o BatchMode=yes -o PreferredAuthentications=password \
  -o PubkeyAuthentication=no user@<target> true 2>&1 \
  | sed -n 's/^debug1: Authentications that can continue: //p'
```

## Subtopics

- **[Default credentials](default-credentials.md)**: vendor and factory SSH logins.
- **[Password brute force](password-brute-force.md)**: online password attacks and spraying.
- **[Public key exposure](public-key-exposure.md)**: finding and reusing private keys.
- **[Weak key generation](weak-key-generation.md)**: predictable keys from broken generators.

## References

- [OpenSSH sshd_config authentication options](https://man.openbsd.org/sshd_config)
- [OpenSSH ssh_config authentication preferences](https://man.openbsd.org/ssh_config)
