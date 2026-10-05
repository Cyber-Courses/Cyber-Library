---
title: "Stack overflow: buffer overflows in telnetd"
description: "telnetd has carried stack buffer overflows in its handling of Telnet options and protocol data, notably the encryption-option and telrcv code paths, reachable pre-authentication. On old Unix and embedded builds without stack protections, a crafted option sequence overflows a fixed buffer to execute code as root."
keywords:
  - stack overflow
  - telnetd
  - telrcv
  - encryption option
  - rce
---

# Stack overflow

The Telnet server's parsing of protocol data and options has produced stack buffer overflows, where a fixed-size buffer is filled from attacker-controlled input without a bound. The best-known class is in the encryption-option handling and the `telrcv` input path, reachable before authentication, where an oversized or crafted option sequence overruns a stack buffer. On the old Unix `telnetd` and embedded builds that still run Telnet, which commonly lack stack canaries, non-executable stacks, and ASLR, this overflow is turned into reliable remote code execution, and because `telnetd` runs as root, the result is root.

```bash
# fingerprint the telnetd build to match the specific overflow
nc <target> 23; nmap -p23 -sV <target>
# the exploit sends crafted Telnet option/negotiation data that overflows a stack
# buffer in the pre-auth parsing; on no-mitigation targets, overwrite the return
# address to shellcode/ROP. Build-specific offsets; match version to the advisory.
```

## Exploitation notes

- The high-value property is pre-authentication reach and root privilege: the vulnerable parsing runs before login and `telnetd` is root, so success is unauthenticated root.
- Legacy Unix and embedded devices are the realistic targets because they lack modern mitigations; a modern, mitigated telnetd is far harder and these specific bugs are patched.
- Exploitation is build-specific (offsets, buffer sizes); fingerprint the exact implementation and version and match the advisory.
- Device telnetd (BusyBox and vendor forks) have their own overflow history; treat embedded Telnet as a prime candidate.

## References

- [Historic telnetd encryption-option overflow advisories](https://www.cve.org/)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
