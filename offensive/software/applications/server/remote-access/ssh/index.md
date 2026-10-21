---
title: "SSH: attacking the Secure Shell service"
order: 3
description: "SSH provides encrypted remote administration, commonly on TCP 22. Assess service identity, effective authentication policy, negotiated algorithms, host and certificate trust, implementation-specific advisories, and the forwarding capabilities granted after access."
keywords:
  - ssh
  - openssh
  - port 22
  - key authentication
  - tunneling
---

# SSH

SSH (Secure Shell) is an encrypted remote-administration protocol, commonly listening on TCP 22. Its assessment spans service identification, effective authentication policy, algorithm negotiation, host-key and certificate trust, implementation-specific vulnerabilities, and the forwarding capabilities available after authentication. A banner or advertised legacy algorithm is evidence for further review, not proof of patch status or a practical downgrade. Likewise, successful authentication may lead to a shell, a forced command, a subsystem, or restricted forwarding rather than unrestricted host control.

```bash
# fingerprint the service
nmap -p22 -sV --script ssh2-enum-algos,ssh-hostkey <target>
# authentication-method review requires an explicit username and begins an auth exchange
nmap -p22 --script ssh-auth-methods --script-args 'ssh.user=<user>' <target>
nc <target> 22                                 # banner (SSH-2.0-OpenSSH_x.y)
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: banner, algorithms, host keys, and version-specific user-enumeration checks.
- **[Authentication](authentication/index.md)**: passwords, defaults, keys, and weak keys.
- **[Weak cryptography](weak-cryptography/index.md)**: cipher, MAC, and key-exchange weaknesses.
- **[MITM and trust](mitm-and-trust/index.md)**: host-key and certificate trust abuse.
- **[Tunneling](tunneling/index.md)**: port forwarding, SOCKS, jump hosts, and agent hijacking.

## References

- [OpenSSH](https://www.openssh.com/manual.html)
- [RFC 4251-4254 (SSH architecture and protocols)](https://datatracker.ietf.org/doc/html/rfc4251)
- [HackTricks: SSH (22)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
