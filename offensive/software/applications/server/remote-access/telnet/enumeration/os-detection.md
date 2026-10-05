---
title: "OS detection: identifying the system behind Telnet"
description: "Beyond the banner, the Telnet login prompt format, option negotiation, and post-login messages fingerprint the operating system: a Unix login:/Password: sequence, a Windows prompt, a Cisco or BusyBox interface, or a vendor menu. Knowing the OS directs credential, exploitation, and post-access choices."
keywords:
  - os detection
  - login prompt
  - option negotiation
  - fingerprint
  - telnet
---

# OS detection

Telnet discloses the operating system through more than the banner. The login sequence differs by OS (`login:`/`Password:` on Unix, distinct Windows prompts, a Cisco `Username:`/`Password:` or enable prompt, a BusyBox shell on embedded Linux, or a vendor configuration menu), the Telnet option negotiation (the IAC command exchange at connect) varies by implementation, and post-login messages (motd, shell type) confirm the platform. Pinning the OS and device type directs which credentials to try, which `telnetd` flaws apply, and what post-access actions make sense.

```bash
nc <target> 23                                 # observe prompt style and any menu
nmap -p23 -O --script telnet-encryption <target>   # OS hints + option negotiation
# the IAC option negotiation bytes at connect differ per implementation
# Unix: "login:" then "Password:"; Cisco: "Username:"/enable; BusyBox: direct shell/menu
```

## Exploitation notes

- The prompt style is a quick OS/device tell: a plain `login:` suggests Unix/Linux, a vendor menu suggests an appliance, a Cisco prompt suggests network gear with its own credential and enable model.
- Option-negotiation behaviour and version fingerprint the `telnetd` implementation, refining the [memory-corruption](../memory-corruption/index.md) targeting.
- The identified OS/device selects the [default-credential](../authentication/default-credentials.md) set and shapes post-access expectations (a router enable prompt, a Unix shell, a restricted menu to escape).
- Combine with the [banner](banner-grabbing.md) for a complete fingerprint before authenticating.

## References

- [RFC 854: option negotiation (IAC)](https://datatracker.ietf.org/doc/html/rfc854)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
