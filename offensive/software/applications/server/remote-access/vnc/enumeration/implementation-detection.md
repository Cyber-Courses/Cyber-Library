---
title: "Implementation detection: identifying the VNC server product"
description: "VNC implementations, RealVNC, TightVNC, UltraVNC, TigerVNC, and the platform servers, differ in supported security types, extensions, and version strings, and each has distinct vulnerabilities. Fingerprinting the implementation from the offered security types and behaviour maps the target to its known authentication-bypass and memory-corruption flaws."
keywords:
  - realvnc
  - tightvnc
  - ultravnc
  - fingerprint
  - security type
---

# Implementation detection

The RFB protocol is standard, but implementations diverge in the security types and extensions they offer and in their version strings, and those differences carry vulnerability. RealVNC, TightVNC, UltraVNC, TigerVNC, and the macOS/embedded servers each support different authentication schemes (the Tight security type and its sub-types, VeNCrypt, vendor schemes) and have their own bug histories, notably authentication-bypass flaws in RealVNC and TightVNC. Identifying the implementation narrows the attack to that product's known issues.

```bash
# the offered security types and extensions fingerprint the implementation
nmap -p5900 --script vnc-info <target>         # security types hint at Tight/UltraVNC/VeNCrypt
# behaviour and version strings distinguish products; vendor security types:
#   16 = Tight (TightVNC/TurboVNC), 17 = Ultra, 19 = VeNCrypt, 2 = standard VNC auth
# a connecting client that negotiates Tight sub-auth reveals TightVNC-family servers
```

## Exploitation notes

- Map the implementation to its flaws: RealVNC has a known security-type-handling authentication bypass, and TightVNC has had authentication and memory-safety issues; identifying the product selects which [bypass](../authentication/index.md) to try.
- The offered security types are the main signal: the Tight (16) and Ultra (17) types indicate those families, VeNCrypt (19) indicates TLS-wrapped VNC, and only standard type 2 suggests a basic server.
- Embedded and appliance VNC (on IoT, KVM-over-IP, industrial devices) often runs old, vulnerable builds with default or no passwords; the implementation and version flag these.
- Pair with the [RFB version](banner-grabbing.md) to complete the fingerprint before attacking.

## References

- [RFB security types registry](https://datatracker.ietf.org/doc/html/rfc6143#section-7.1.2)
- [HackTricks: VNC implementations](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
