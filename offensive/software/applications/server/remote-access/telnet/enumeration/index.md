---
title: "Enumeration: fingerprinting a Telnet service"
description: "Telnet enumeration reads the service banner and login prompt, which are often verbose, disclosing the operating system, device model, and software version, and sometimes a pre-login warning or configuration detail. That identification maps the target to default credentials and to the telnetd memory-corruption flaws it may carry."
keywords:
  - telnet enumeration
  - banner
  - os detection
  - login prompt
  - fingerprint
---

# Enumeration

Telnet is talkative: on connect it typically presents a banner and a login prompt, and both frequently disclose the operating system, device vendor and model, and software version, with some systems adding pre-login messages that reveal configuration or purpose. This identification drives the follow-on, the device and OS map directly to known default credentials and to the `telnetd` implementation's memory-corruption vulnerabilities, and the login-prompt format itself can leak OS specifics useful for fingerprinting.

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
