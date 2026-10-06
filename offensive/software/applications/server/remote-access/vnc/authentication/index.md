---
title: "Authentication: attacking VNC access control"
order: 1
description: "VNC access control is frequently weak: servers run with no authentication, with vendor or guessable passwords, or with the classic VNC scheme whose eight-character DES password is brute-forceable. Specific implementations add outright authentication bypasses. Gaining access yields a full interactive desktop on the target."
keywords:
  - vnc authentication
  - no authentication
  - default password
  - brute force
  - auth bypass
---

# Authentication

VNC authentication is weak in several independent ways, and any one gives a full interactive desktop. Servers are commonly left with the "None" security type (no authentication). The classic VNC password scheme is a DES challenge-response over a password truncated to eight characters, which is brute-forceable online and crackable offline from a captured challenge, and such passwords are often vendor defaults or trivial. And particular implementations (RealVNC, TightVNC) have had authentication bypasses that skip the check entirely. The payoff is the same regardless: interactive control of the remote desktop.

```bash
vncviewer <target>::5900                       # prompts (or not) depending on security type
nmap -p5900 --script vnc-info,realvnc-auth-bypass <target>
```

## Subtopics

- **[No authentication](no-authentication.md)**: servers offering the None security type.
- **[Default passwords](default-passwords.md)**: vendor and guessable VNC passwords.
- **[Password brute force](password-brute-force.md)**: online and offline attacks on the VNC password.
- **[RealVNC auth bypass](realvnc-auth-bypass.md)**: the RealVNC security-type bypass.
- **[TightVNC bypass](tightvnc-bypass.md)**: TightVNC authentication flaws.

## References

- [RFC 6143: security](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
