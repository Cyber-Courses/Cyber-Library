---
title: "Memory corruption: remote code execution in telnetd"
description: "Telnet server implementations have carried serious memory-corruption vulnerabilities reachable before or during authentication: stack buffer overflows and format-string flaws in option and environment handling. On old Unix telnetd and embedded builds, these give remote code execution, often as root, against a service that predates modern mitigations."
keywords:
  - telnetd
  - buffer overflow
  - format string
  - pre-auth rce
  - memory corruption
---

# Memory corruption

Beyond its protocol weaknesses, the Telnet server code itself has a long history of memory-corruption bugs, and because `telnetd` often runs as root and predates modern exploit mitigations (especially on legacy Unix and embedded builds), these yield remote code execution with high privilege. The classic classes are stack buffer overflows in option and environment handling (the `telrcv`/encryption-option code paths) and format-string flaws, several reachable pre-authentication. An exposed old `telnetd` is therefore not just a weak login but a direct RCE target once fingerprinted.

```bash
# fingerprint the telnetd implementation/version (banner, option negotiation)
nc <target> 23; nmap -p23 -sV <target>
# match the build to the applicable advisory: stack overflow vs format string
```

## Subtopics

- **[Stack overflow](stack-overflow.md)**: buffer overflows in option/environment handling.
- **[Format string](format-string.md)**: format-string flaws in telnetd.

## References

- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [Historic telnetd advisories](https://www.cve.org/)
