---
title: "Implementation detection: identifying the VNC server product"
description: "RFB security types, extensions, banners, and behavior can suggest a VNC implementation, but shared extensions are not definitive product identity. Corroborate network hints with endpoint, package, or service evidence before matching a named advisory."
keywords:
  - realvnc
  - tightvnc
  - ultravnc
  - fingerprint
  - security type
---

# Implementation detection

RFB implementations differ in supported security types, extensions, default settings, and version reporting. Those traits can suggest RealVNC, TightVNC, UltraVNC, TigerVNC, a platform server, or an embedded fork, but compatible products can implement the same registered type. Treat network observations as fingerprinting evidence with stated confidence. Where possible, corroborate them with process names, package inventory, binary metadata, service definitions, or appliance firmware information before matching a named advisory.

```bash
# the offered security types and extensions fingerprint the implementation
nmap -p5900 --script vnc-info <target>         # security types hint at Tight/UltraVNC/VeNCrypt
# behaviour and version strings distinguish products; vendor security types:
#   16 = Tight (TightVNC/TurboVNC), 17 = Ultra, 19 = VeNCrypt, 2 = standard VNC auth
# shared extensions are hints; confirm product and version through endpoint evidence
```

## Exploitation notes

- Registered types such as Tight (16), Ultra (17), TLS (18), and VeNCrypt (19) are useful hints, not proof of a particular product or vulnerable build.
- A server offering only VNC Authentication (2) is not necessarily a specific "basic" implementation; many products retain compatibility with it.
- Record the complete product, platform, and version before evaluating an advisory such as [CVE-2006-2369](../authentication/realvnc-auth-bypass.md).
- Pair network observations with the [RFB version](banner-grabbing.md) and endpoint evidence, preserving uncertainty when exact identification is unavailable.

## References

- [Maintained RFB security-type registry](https://github.com/rfbproto/rfbproto/blob/master/rfbproto.rst)
- [Nmap vnc-info](https://nmap.org/nsedoc/scripts/vnc-info.html)
