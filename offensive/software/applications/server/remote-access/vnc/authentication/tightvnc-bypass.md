---
title: "TightVNC bypass: authentication and memory-safety flaws"
description: "TightVNC and its derivatives have carried authentication-handling and memory-corruption vulnerabilities, including flaws in the Tight security-type sub-authentication negotiation and buffer handling that allow bypassing the password check or crashing and exploiting the server. Fingerprinting the TightVNC version maps a target to the applicable flaw."
keywords:
  - tightvnc
  - tight security type
  - auth bypass
  - memory corruption
  - rfb
---

# TightVNC bypass

TightVNC (and derivatives such as TurboVNC and some embedded forks) introduces the Tight security type, which adds a sub-authentication negotiation on top of the base RFB handshake, and its code has carried both authentication-handling flaws and memory-safety bugs. The authentication issues allow influencing or skipping the sub-auth step to reach a session without valid credentials in affected versions; the memory-corruption issues (in message and encoding handling) allow crashing or exploiting the server. The right attack depends on the exact TightVNC version, so fingerprinting is the first step.

```bash
# identify TightVNC and its version via the offered Tight security type and behaviour
nmap -p5900 --script vnc-info <target>         # Tight (16) security type indicates the family
# match the version to the applicable advisory: auth-handling bypass vs memory-corruption
# auth path: manipulate the Tight sub-auth negotiation to avoid the password check
# memory path: send crafted messages/encodings that the server mishandles
```

## Exploitation notes

- The Tight security type's extra sub-authentication step is the distinctive surface; version-specific flaws there allow reaching a session without the password, similar in spirit to the RealVNC bypass but in the Tight negotiation.
- Memory-corruption bugs in TightVNC's handling of RFB messages and encodings are a separate class, giving crashes or code execution against the server; these are build-specific.
- Embedded and appliance devices often bundle old TightVNC forks with unpatched versions of these flaws plus weak/default passwords; fingerprint the [implementation](../enumeration/implementation-detection.md) and version.
- Where no bypass applies, the password scheme is still weak, see [Password brute force](password-brute-force.md) and [Default passwords](default-passwords.md).

## References

- [TightVNC security advisories](https://www.tightvnc.com/)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
