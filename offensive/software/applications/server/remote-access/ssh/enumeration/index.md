---
title: "Enumeration: fingerprinting an SSH service"
order: 3
description: "SSH enumeration records the self-reported identification string, advertised algorithms, and host keys. Authentication-method and username probes begin authentication exchanges, may be logged, and provide configuration- or version-specific evidence rather than universal account validation."
keywords:
  - ssh enumeration
  - banner
  - algorithms
  - user enumeration
  - host key
---

# Enumeration

SSH exposes an identification string before key exchange and advertises algorithm lists during negotiation. These can suggest an implementation, package lineage, and transport configuration, while host-key fingerprints help correlate endpoints. They do not prove patch status or a practical cryptographic attack. Authentication-method and username probes go further: they initiate authentication, are likely to be logged, and must use an explicit username. Username validation requires a documented implementation- and version-specific signal.

```bash
nmap -p22 -sV --script ssh2-enum-algos,ssh-hostkey <target>
# intrusive: begins authentication for the supplied identity
nmap -p22 --script ssh-auth-methods --script-args 'ssh.user=<user>' <target>
nc <target> 22                                 # raw banner
```

## Subtopics

- **[Banner grabbing](banner-grabbing.md)**: self-reported version and implementation hints.
- **[Algorithm enumeration](algorithm-enumeration.md)**: offered crypto and weak-option detection.
- **[User enumeration](user-enumeration.md)**: version-specific account-disclosure behavior.

## References

- [Nmap ssh-auth-methods](https://nmap.org/nsedoc/scripts/ssh-auth-methods.html)
- [Nmap ssh2-enum-algos](https://nmap.org/nsedoc/scripts/ssh2-enum-algos.html)
