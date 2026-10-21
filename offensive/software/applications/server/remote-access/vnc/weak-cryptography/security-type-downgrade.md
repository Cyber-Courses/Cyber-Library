---
title: "Security type downgrade: forcing a weaker VNC security type"
description: "In RFB 3.7 and later the server offers a list of security types and the client chooses one. When the server offers both strong and weak types, a positioned attacker may strip the strong choices from the unauthenticated list and steer a compatible client toward a weaker type that the server already accepts."
keywords:
  - security type downgrade
  - rfb 3.7
  - negotiation
  - none
  - mitm
---

# Security type downgrade

From RFB protocol 3.7 onward, the server sends a list of supported security types and the client selects one. That initial list is unauthenticated and modifiable in transit. If the server offers both a strong type such as VeNCrypt and a weaker type such as VNC Authentication or None, a machine-in-the-middle can remove the strong choices from the list delivered to the client. A compatible client may then select a weak type that the server already offered and will accept. The attacker cannot invent a type the server does not support: a server configured to offer only strong types will reject an unoffered selection rather than silently accept the downgrade.

```bash
# from an on-path position, intercept the RFB handshake and rewrite the security-type
# list the server offers, removing VeNCrypt(19)/strong types while retaining a weak type
# that the server genuinely offered, such as None(1) or VNC Authentication(2)
# (combine with ARP/DNS/route positioning; the 3.3 protocol has the server dictate the
#  single type, so downgrade applies to 3.7+ where the client chooses from a list)
```

## Exploitation notes

- The downgrade opportunity exists only when the server offers strong and weak security types together; the attacker edits the client-facing list but must leave a type the server accepts.
- Removing VeNCrypt/TLS can steer a compatible client to offered VNC Authentication or None, re-exposing the session to [cleartext](cleartext-transmission.md) capture and [password cracking](../authentication/password-brute-force.md).
- It requires an on-path position and applies to RFB 3.7+ (where the client picks from a list); RFB 3.3 servers dictate the single type, so there is nothing to downgrade there, only to intercept.
- A server that offers only VeNCrypt or another strong type is not bypassed merely by rewriting the list; the attempted selection fails.

## References

- [RFC 6143: protocol 3.7 security-type negotiation](https://datatracker.ietf.org/doc/html/rfc6143#section-7.1.2)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
