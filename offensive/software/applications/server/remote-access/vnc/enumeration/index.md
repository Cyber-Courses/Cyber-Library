---
title: "Enumeration: fingerprinting a VNC service"
order: 3
description: "VNC enumeration reads the RFB protocol version and the security types the server offers during the unauthenticated handshake, and identifies the implementation (RealVNC, TightVNC, UltraVNC). The security types reveal whether the server requires no authentication or the weak VNC password scheme, and the implementation maps to specific authentication-bypass flaws."
keywords:
  - vnc enumeration
  - rfb version
  - security types
  - implementation
  - vnc-info
---

# Enumeration

VNC's handshake is informative before any authentication: the server announces its RFB protocol version and the list of security types it supports, and the implementation can be fingerprinted from version strings and behaviour. The security-type list is the key finding, it shows whether the server offers "None" (no authentication), the weak VNC DES password scheme, or vendor schemes, and the implementation (RealVNC, TightVNC, UltraVNC) maps to known authentication-bypass vulnerabilities, so enumeration directs the whole attack.

```bash
nmap -p5900 --script vnc-info <target>         # RFB version + supported security types
# raw handshake: the server sends "RFB 003.00X" then the security-type list
nc <target> 5900 | head -c 12                  # RFB protocol version banner
```

## Subtopics

- **[Banner grabbing](banner-grabbing.md)**: the RFB version string.
- **[Implementation detection](implementation-detection.md)**: identifying RealVNC, TightVNC, UltraVNC.

## References

- [RFC 6143: protocol version and security types](https://datatracker.ietf.org/doc/html/rfc6143)
- [nmap vnc-info](https://nmap.org/nsedoc/scripts/vnc-info.html)
