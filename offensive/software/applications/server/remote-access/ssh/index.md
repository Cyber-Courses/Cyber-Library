---
title: "SSH: attacking the Secure Shell service"
order: 3
description: "SSH on TCP 22 provides encrypted remote administration, so it resists passive attack but exposes a rich active surface: service and user enumeration, password and key authentication attacks, weak-crypto negotiation and downgrade, host-key and certificate trust abuse for machine-in-the-middle, and tunnelling that turns a single SSH foothold into network pivoting."
keywords:
  - ssh
  - openssh
  - port 22
  - key authentication
  - tunneling
---

# SSH

SSH (Secure Shell) is the standard encrypted remote-administration protocol, listening on TCP 22, and because the session is encrypted and integrity-protected the attack surface is active rather than passive. Five aspects matter. Enumeration fingerprints the version, supported algorithms, and valid users. Authentication is attacked through passwords, default credentials, and the keys that really grant access. Weak cryptography lets a positioned attacker negotiate or downgrade to breakable ciphers, MACs, or key exchange. Host-key and certificate trust, built on trust-on-first-use, is abused for machine-in-the-middle. And SSH's tunnelling features turn one reachable host into a pivot across the network.

```bash
# fingerprint the service
nmap -p22 -sV --script ssh2-enum-algos,ssh-hostkey,ssh-auth-methods <target>
nc <target> 22                                 # banner (SSH-2.0-OpenSSH_x.y)
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: banner, algorithms, and valid users.
- **[Authentication](authentication/index.md)**: passwords, defaults, keys, and weak keys.
- **[Weak cryptography](weak-cryptography/index.md)**: cipher, MAC, and key-exchange weaknesses.
- **[MITM and trust](mitm-and-trust/index.md)**: host-key and certificate trust abuse.
- **[Tunneling](tunneling/index.md)**: port forwarding, SOCKS, jump hosts, and agent hijacking.

## References

- [OpenSSH](https://www.openssh.com/manual.html)
- [RFC 4251-4254 (SSH architecture and protocols)](https://datatracker.ietf.org/doc/html/rfc4251)
- [HackTricks: SSH (22)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
