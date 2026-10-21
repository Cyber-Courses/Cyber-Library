---
title: "Memory corruption: remote code execution in telnetd"
order: 2
description: "Specific Telnet server implementations have carried serious memory-corruption vulnerabilities. CVE-2011-4862 affected named BSD-derived telnetd implementations that supported the encryption option and allowed pre-authentication code execution with the daemon's privileges."
keywords:
  - telnetd
  - buffer overflow
  - CVE-2011-4862
  - pre-auth rce
  - memory corruption
---

# Memory corruption

Beyond its protocol weaknesses, particular Telnet server implementations have contained memory-corruption vulnerabilities. One well-documented example is CVE-2011-4862 in the encryption-option code shared by affected FreeBSD, Heimdal, MIT Kerberos applications, and GNU inetutils releases. An unauthenticated client could supply an overlong encryption key that was copied into a fixed-size buffer, enabling code execution with the daemon's privileges. Exposure to Telnet or support for encryption does not by itself prove vulnerability: identify the implementation and affected version before drawing that conclusion.

```bash
# fingerprint the telnetd implementation/version (banner, option negotiation)
nc <target> 23; nmap -p23 -sV <target>
# compare the implementation and version with a specific vendor advisory
```

## Subtopics

- **[Encryption-key buffer overflow](stack-overflow.md)**: CVE-2011-4862 in affected BSD-derived telnetd code.

## References

- [FreeBSD-SA-11:08.telnetd](https://www.freebsd.org/security/advisories/FreeBSD-SA-11:08.telnetd.asc)
- [Nmap telnet-encryption](https://nmap.org/nsedoc/scripts/telnet-encryption.html)
