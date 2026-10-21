---
title: "Enumeration: fingerprinting a VNC service"
order: 3
description: "VNC enumeration records the RFB protocol version and advertised security types, then uses extensions, behavior, and endpoint evidence to estimate the implementation. Security types describe available mechanisms but do not alone prove product identity or vulnerability."
keywords:
  - vnc enumeration
  - rfb version
  - security types
  - implementation
  - vnc-info
---

# Enumeration

VNC's handshake exposes useful information before authentication. The server first announces its RFB protocol version; after the client replies, security negotiation identifies the types the server supports. The list can show `None`, legacy VNC Authentication, TLS, VeNCrypt, SASL, or vendor extensions. These values describe available mechanisms and can suggest an implementation, but exact product and version identification normally requires corroborating endpoint, package, service, or firmware evidence. Match vulnerabilities only after that identification is sufficiently reliable.

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
