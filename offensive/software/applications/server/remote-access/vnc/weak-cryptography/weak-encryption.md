---
title: "Weak encryption: VNC optional and weak encryption modes"
description: "VNC encryption depends on implementation and configuration. VeNCrypt can provide TLS, while deployments that also offer cleartext security types, use obsolete TLS settings, or fail to validate certificates retain avoidable interception risks."
keywords:
  - weak encryption
  - venCrypt
  - proprietary encryption
  - optional encryption
  - tls
---

# Weak encryption

Because base VNC is cleartext, implementations have added several security types, including VeNCrypt, which negotiates a TLS-protected stream after the initial RFB and VeNCrypt handshakes. Protection depends on the selected subtype, TLS policy, and client certificate validation. A deployment that offers both encrypted and cleartext security types permits clients to choose the weaker option and may expose the unauthenticated selection to an on-path downgrade. The presence of an encryption option therefore does not prove that every session uses it.

```bash
# what encryption/security types are offered, and are they required?
nmap -p5900 --script vnc-info <target>         # VeNCrypt(19)/vendor types present?
# compare the complete list: VeNCrypt alongside None/VNC Auth means encryption is optional
```

## Exploitation notes

- The key question is whether encryption is required or merely offered; when a weaker type is also offered, a client can select it and an on-path attacker may attempt a [security-type downgrade](security-type-downgrade.md).
- A generic `openssl s_client` connection to port 5900 cannot assess VeNCrypt because TLS begins only after RFB version, security-type, VeNCrypt-version, and subtype negotiation. Use a VeNCrypt-aware client and inspect its certificate and TLS diagnostics.
- VeNCrypt/TLS is only as strong as its TLS configuration and certificate validation; accepting an untrusted certificate enables interception from a suitable network position.
- Where no effective encryption is in force, the session is as exposed as [cleartext transmission](cleartext-transmission.md).

## References

- [RFB protocol and VeNCrypt security types](https://github.com/rfbproto/rfbproto/blob/master/rfbproto.rst)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
