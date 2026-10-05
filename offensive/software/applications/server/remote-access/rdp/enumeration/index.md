---
title: "Enumeration: fingerprinting an RDP service"
description: "RDP enumeration gathers the host's identity and security posture before attacking: the version and, via the NTLM handshake, the hostname and domain, and the security settings, whether Network Level Authentication is required, the encryption level, and the TLS certificate. These determine which authentication, exposure, and pre-auth paths apply."
keywords:
  - rdp enumeration
  - rdp-ntlm-info
  - nla
  - encryption level
  - certificate
---

# Enumeration

Enumerating RDP establishes what you are attacking and how it is configured. The service leaks useful identity data, notably the hostname, domain, and OS build through the NTLM security provider during the connection handshake, and its security settings decide the attack: whether Network Level Authentication (NLA) is required (which forces credentials before a session), the negotiated security and encryption level, and the TLS certificate (which names the host and dates the system). This drives the choice between brute force, exposure analysis, and pre-auth exploitation.

```bash
nmap -p3389 --script rdp-ntlm-info <target>    # hostname, domain, OS build via NTLM
nmap -p3389 --script rdp-enum-encryption <target>   # security layer and encryption level
```

## Subtopics

- **[Banner grabbing](banner-grabbing.md)**: version and identity disclosure.
- **[Security settings](security-settings.md)**: NLA, encryption level, and certificate posture.

## References

- [nmap rdp-ntlm-info](https://nmap.org/nsedoc/scripts/rdp-ntlm-info.html)
- [HackTricks: RDP enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
