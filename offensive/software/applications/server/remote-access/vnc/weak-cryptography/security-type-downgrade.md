---
title: "Security type downgrade: forcing a weaker VNC security type"
description: "In RFB 3.7 and later the server offers a list of security types and the client chooses one. A positioned attacker intercepting the handshake strips the strong types from the server's list or forces the client to pick None or the weak VNC auth, downgrading the session to no or weak authentication and defeating stronger configured options."
keywords:
  - security type downgrade
  - rfb 3.7
  - negotiation
  - none
  - mitm
---

# Security type downgrade

From RFB protocol 3.7 onward, the server sends a list of supported security types and the client selects one. That negotiation is unauthenticated and, on an unencrypted channel, modifiable in transit, which enables a downgrade. A machine-in-the-middle attacker strips the strong types (VeNCrypt/TLS, or even VNC auth) from the server's offered list before it reaches the client, leaving only None or the weak VNC password scheme, so the client proceeds with no or weak authentication. This defeats a server that was configured to require stronger security, because nothing authenticates the type list.

```bash
# from an on-path position, intercept the RFB handshake and rewrite the security-type
# list the server offers, removing VeNCrypt(19)/strong types, leaving None(1) or VNC(2)
# the client then negotiates the weak/none type, and the session proceeds unprotected
# (combine with ARP/DNS/route positioning; the 3.3 protocol has the server dictate the
#  single type, so downgrade applies to 3.7+ where the client chooses from a list)
```

## Exploitation notes

- The downgrade works because the security-type negotiation is neither encrypted nor integrity-protected on a cleartext channel; the attacker edits the offered list in transit.
- Removing VeNCrypt/TLS from the list drops the session to unencrypted VNC auth or None, re-exposing it to [cleartext](cleartext-transmission.md) capture and [password cracking](../authentication/password-brute-force.md).
- It requires an on-path position and applies to RFB 3.7+ (where the client picks from a list); RFB 3.3 servers dictate the single type, so there is nothing to downgrade there, only to intercept.
- This is the active complement to passively reading a cleartext session: it forces weakness where the server offered strength.

## References

- [RFC 6143: protocol 3.7 security-type negotiation](https://datatracker.ietf.org/doc/html/rfc6143#section-7.1.2)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
