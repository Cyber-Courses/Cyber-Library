---
title: "MITM and trust: abusing SSH host-key and certificate trust"
order: 4
description: "SSH authenticates the server by its host key, trusted on first use and pinned in known_hosts, or signed by a CA for certificate-based SSH. Where that trust is weak, a positioned attacker can perform a machine-in-the-middle attack to capture passwords, observe or alter sessions, and abuse explicitly forwarded agents."
keywords:
  - ssh mitm
  - host key
  - known_hosts
  - trust on first use
  - certificate
---

# MITM and trust

SSH resists machine-in-the-middle attacks because the client verifies the server's host key: on first connection the key may be recorded in `known_hosts` (trust on first use), and later connections must match. Certificate-based SSH replaces per-host pinning with trust in a CA that signs host keys. The attack surface is where that trust is weak: a client configured to accept unknown or changed keys, a first connection whose fingerprint is not verified out of band, a poisoned `known_hosts`, or compromise of a trusted host CA. A positioned attacker can then interpose, present a substitute host key, capture password authentication, and observe or alter the plaintext session. Public-key authentication does not transmit the private key; onward use of that identity requires another capability, such as access to an explicitly forwarded agent.

```bash
# position (ARP/DNS/route) + an SSH MITM proxy that presents a substitute host key
# the attack succeeds when the client does not reject the unknown/changed key
```

## Subtopics

- **[Known-hosts bypass](known-hosts-bypass.md)**: defeating trust-on-first-use and key pinning.
- **[SSH MITM](ssh-mitm.md)**: interposing to capture credentials and hijack the session.
- **[Certificate trust abuse](certificate-validation-bypass.md)**: abusing compromised or over-broad certificate-authority trust.

## References

- [OpenSSH known_hosts and host keys](https://man.openbsd.org/sshd.8)
- [HackTricks: SSH MITM](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
