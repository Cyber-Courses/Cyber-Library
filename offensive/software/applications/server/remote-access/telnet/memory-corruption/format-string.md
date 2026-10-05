---
title: "Format string: format-string flaws in telnetd"
description: "telnetd and related code have carried format-string vulnerabilities where attacker-controlled input, such as environment or terminal data passed through Telnet, reaches a printf-family function as the format argument. This yields memory read and write, and on vulnerable builds remote code execution, typically as root."
keywords:
  - format string
  - telnetd
  - printf
  - memory write
  - rce
---

# Format string

Format-string bugs occur when attacker-controlled data is passed as the format argument to a `printf`-family function, letting the attacker use format specifiers (`%x`, `%n`) to read from and write to memory. Telnet has exposed this class where client-supplied data, environment variables, terminal type, or option values forwarded by `telnetd` into logging or processing, reaches a formatting function unsanitised. The `%n` specifier gives an arbitrary write, which on a vulnerable build is escalated to control flow and remote code execution, again typically as root given `telnetd`'s privilege.

```bash
# fingerprint to match the specific format-string flaw
nc <target> 23; nmap -p23 -sV <target>
# the exploit supplies format specifiers in the vulnerable field (e.g. an option/
# environment value telnetd logs or processes with printf-family code):
#   %x...%x to leak stack/memory, %n to write; chain the write to hijack control flow.
# offsets are build-specific; match version to the advisory.
```

## Exploitation notes

- Format-string flaws give both a read (leak, useful to defeat ASLR where present) and a write (`%n`), so they are flexible primitives that can self-bootstrap an exploit on mitigated and unmitigated targets alike.
- The vulnerable input is whatever client-controlled field reaches a formatting function; on Telnet that has included forwarded environment/terminal data and logging paths.
- As with the overflows, legacy and embedded telnetd running as root are the realistic targets, and exploitation is build-specific.
- Where no memory-corruption bug applies, Telnet's cleartext and weak-auth surfaces remain; this class is the direct-RCE path when the build is vulnerable.

## References

- [Historic telnetd format-string advisories](https://www.cve.org/)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
