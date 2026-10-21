---
title: "VNC: attacking Virtual Network Computing"
order: 6
description: "VNC shares a graphical desktop over RFB, commonly on TCP 5900 plus the display number. Security depends on the selected RFB security type, implementation, version, password policy, transport protection, and server configuration."
keywords:
  - vnc
  - rfb protocol
  - port 5900
  - des challenge
  - remote desktop
---

# VNC

VNC (Virtual Network Computing) shares a graphical desktop using the Remote Framebuffer (RFB) protocol, commonly on TCP 5900 plus the display number. RFB supports multiple security types, and implementations add further extensions. `None` and legacy VNC Authentication do not encrypt subsequent protocol data; the latter uses a DES challenge-response and only the first eight password bytes. Other types can add TLS, SASL, RSA-AES, SSH, or vendor-specific protection. Evaluate the security types actually offered and selected, the password and access policy, and the exact implementation and version. Treat vulnerabilities as advisory-specific findings, such as CVE-2006-2369 in RealVNC 4.1.0–4.1.1.

```bash
nmap -p5900-5902 --script vnc-info,vnc-title,realvnc-auth-bypass <target>
vncviewer <target>::5900                       # connect to test auth/no-auth
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: RFB version, security types, and implementation.
- **[Authentication](authentication/index.md)**: security types, legacy passwords, configuration, and advisory-specific bypasses.
- **[Transport security](weak-cryptography/index.md)**: selected cleartext or protected modes and conditional downgrade risk.

## References

- [RFC 6143 (RFB protocol)](https://datatracker.ietf.org/doc/html/rfc6143)
- [Maintained RFB protocol specification](https://github.com/rfbproto/rfbproto/blob/master/rfbproto.rst)
