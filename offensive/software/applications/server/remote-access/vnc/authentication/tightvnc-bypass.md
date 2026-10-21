---
title: "TightVNC advisory and configuration assessment"
description: "The Tight security type identifies a protocol extension, not an authentication bypass. Record the exact implementation and version, then require a named advisory, affected range, prerequisites, and demonstrated impact before claiming authentication bypass or memory corruption."
keywords:
  - tightvnc
  - tight security type
  - advisory matching
  - configuration assessment
  - rfb
---

# TightVNC advisory and configuration assessment

The Tight security type extends RFB with capability and authentication negotiation. Seeing it in a server handshake can support implementation identification, but does not establish that the server is vulnerable. TightVNC, TurboVNC, and embedded forks have separate code histories and release practices, so vulnerability claims cannot be transferred among them without evidence. Record the exact product, platform, and version, then compare them with a named vendor advisory or CVE.

```bash
# enumerate advertised RFB security types
nmap -sV -p5900 --script vnc-info <target>
# confirm the implementation and version through endpoint/package evidence where possible
```

## Exploitation notes

- Do not describe the Tight security type itself as a bypass or vulnerable condition.
- Do not group TightVNC, TurboVNC, and unidentified embedded forks under one affected range.
- For every vulnerability claim, cite the advisory and preserve its platform, affected versions, attack vector, and prerequisites.
- Assess configured security types separately; legacy VNC Authentication has the limitations described in [Password brute force](password-brute-force.md).

## References

- [RFC 6143 security types](https://datatracker.ietf.org/doc/html/rfc6143#section-7.2)
- [TightVNC](https://www.tightvnc.com/)
