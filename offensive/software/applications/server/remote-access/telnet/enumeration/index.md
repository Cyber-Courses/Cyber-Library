---
title: "Enumeration: fingerprinting a Telnet service"
order: 3
description: "Telnet enumeration records protocol negotiation, any service banner, and the login prompt. These may reveal product or operating-system clues, but exact model, version, credentials, and vulnerability status require corroboration."
keywords:
  - telnet enumeration
  - banner
  - os detection
  - login prompt
  - fingerprint
---

# Enumeration

A Telnet connection may present protocol negotiation, a banner, and a login prompt. Those responses can reveal an operating-system family, device vendor, model, software version, or operational purpose, but many are generic, suppressed, or customized. Treat banner-derived identity as a hypothesis to corroborate with other authorized evidence. An exact product and version can guide a check of applicable credential policy and vendor advisories; it does not by itself prove that a default credential works or that a vulnerability is present.

```bash
nc <target> 23                                 # banner + login prompt
nmap -p23 -sV <target>                         # service/version
nmap -p23 --script telnet-ntlm-info <target>   # NTLM info on Windows telnet
```

## Subtopics

- **[Banner grabbing](banner-grabbing.md)**: the service banner and version.
- **[OS detection](os-detection.md)**: identifying the OS from banner and prompt.

## References

- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
