---
title: "Authentication: attacking VNC access control"
order: 1
description: "VNC authentication depends on the security types and implementation in use. Review whether None or legacy VNC Authentication is enabled, assess password policy and rate limiting, and match implementation vulnerabilities to a named advisory and affected version."
keywords:
  - vnc authentication
  - no authentication
  - default password
  - brute force
  - auth bypass
---

# Authentication

RFB supports multiple security types, and implementations add further authentication mechanisms. A server offering `None` permits a connection without RFB authentication. Legacy VNC Authentication uses a DES challenge-response and, in implementations that follow the classic scheme, uses only the first eight password bytes. Online guessing depends on password quality and server-side attempt controls; a captured challenge-response permits offline verification of candidates. Implementation vulnerabilities must be tied to an exact advisory, such as CVE-2006-2369 in RealVNC 4.1.0 and 4.1.1, rather than inferred from the product family.

```bash
vncviewer <target>::5900                       # prompts (or not) depending on security type
nmap -p5900 --script vnc-info,realvnc-auth-bypass <target>
```

## Subtopics

- **[No authentication](no-authentication.md)**: servers offering the None security type.
- **[Default passwords](default-passwords.md)**: vendor and guessable VNC passwords.
- **[Password brute force](password-brute-force.md)**: online and offline attacks on the VNC password.
- **[RealVNC auth bypass](realvnc-auth-bypass.md)**: the RealVNC security-type bypass.
- **[TightVNC advisory assessment](tightvnc-bypass.md)**: requiring evidence for version-specific claims.

## References

- [RFC 6143: security](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
