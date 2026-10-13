---
title: "MITM and trust: abusing SSH host-key and certificate trust"
order: 4
description: "SSH authenticates the server by its host key, trusted on first use and pinned in known_hosts, or signed by a CA for certificate-based SSH. Where that trust is weak, users who ignore host-key warnings, clients set to accept any key, or certificate validation flaws, a positioned attacker performs a machine-in-the-middle to capture credentials and hijack the session."
keywords:
  - ssh mitm
  - host key
  - known_hosts
  - trust on first use
  - certificate
---

# MITM and trust

SSH resists machine-in-the-middle only because the client verifies the server's host key: on first connection the key is recorded in `known_hosts` (trust on first use), and later connections must match or the client warns. Certificate-based SSH replaces this with a CA that signs host keys. The attack surface is where that trust is weak: users conditioned to accept host-key-changed warnings, clients configured to accept any key (`StrictHostKeyChecking no`), fresh connections with no pinned key yet, and flaws in certificate validation. A positioned attacker exploits these to interpose, presenting their own host key, decrypting the session, capturing the password or key, and relaying to the real server.

```bash
# position (ARP/DNS/route) + an SSH MITM proxy that presents a substitute host key
# the attack succeeds when the client does not reject the unknown/changed key
```

## Subtopics

- **[Known-hosts bypass](known-hosts-bypass.md)**: defeating trust-on-first-use and key pinning.
- **[SSH MITM](ssh-mitm.md)**: interposing to capture credentials and hijack the session.
- **[Certificate validation bypass](certificate-validation-bypass.md)**: abusing certificate-based SSH trust.

## References

- [OpenSSH known_hosts and host keys](https://man.openbsd.org/sshd.8)
- [HackTricks: SSH MITM](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
