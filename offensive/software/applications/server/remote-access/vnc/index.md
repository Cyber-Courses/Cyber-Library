---
title: "VNC: attacking Virtual Network Computing"
description: "VNC shares a graphical desktop over the RFB protocol (typically ports 5900+). Its weaknesses are a short challenge-response password scheme capped at eight characters, servers left with no authentication, cleartext screen and input traffic, a negotiable security type that can be downgraded, and implementation-specific authentication bypasses in RealVNC and TightVNC."
keywords:
  - vnc
  - rfb protocol
  - port 5900
  - des challenge
  - remote desktop
---

# VNC

VNC (Virtual Network Computing) shares a graphical desktop over the Remote Framebuffer (RFB) protocol, usually on TCP 5900 and up (display N on 5900+N). Its security is weak by design and by deployment. The classic VNC authentication is a DES challenge-response using a password truncated to eight characters, trivial to brute force or crack from a captured challenge. Many servers run with no authentication at all. The screen contents and keystrokes travel in cleartext unless tunnelled. The RFB security type is negotiated and can be downgraded. And specific implementations (RealVNC, TightVNC) have shipped outright authentication bypasses.

```bash
nmap -p5900-5902 --script vnc-info,vnc-title,realvnc-auth-bypass <target>
vncviewer <target>::5900                       # connect to test auth/no-auth
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: RFB version, security types, and implementation.
- **[Authentication](authentication/index.md)**: no-auth, defaults, brute force, and bypasses.
- **[Weak cryptography](weak-cryptography/index.md)**: cleartext, weak encryption, and downgrade.

## References

- [RFC 6143 (RFB protocol)](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC (5900)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
