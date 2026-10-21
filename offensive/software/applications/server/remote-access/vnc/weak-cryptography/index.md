---
title: "Transport security: VNC security types and encryption"
order: 2
description: "Standard VNC does not encrypt the session, so screen contents, keystrokes, and the authentication challenge-response cross the network in cleartext. When a server offers weak and strong RFB security types together, the unauthenticated selection may also permit downgrade to a weak type."
keywords:
  - vnc encryption
  - cleartext
  - rfb security type
  - downgrade
  - weak crypto
---

# Transport security

The `None` and VNC Authentication security types send subsequent RFB protocol data without encryption. Other registered or vendor mechanisms can add TLS, SASL protection, RSA-AES, SSH tunneling, or other security properties. Determine exposure from the type and subtype selected for the actual session, including certificate or peer validation where applicable. When a server offers both protected and cleartext types, unauthenticated negotiation may permit an on-path attacker to steer a compatible client toward a weaker type the server already accepts; a server offering only protected types is not bypassed by inventing another selection.

```bash
# inventory advertised security types; this alone does not prove which type a session uses
nmap -p5900 --script vnc-info <target>
tcpdump -i eth0 -w vnc.pcap port 5900          # capture for analysis
```

## Subtopics

- **[Cleartext transmission](cleartext-transmission.md)**: reading the unencrypted desktop and credentials.
- **[Weak encryption](weak-encryption.md)**: weak or optional encryption modes.
- **[Security type downgrade](security-type-downgrade.md)**: forcing a weaker security type.

## References

- [RFC 6143: security types](https://datatracker.ietf.org/doc/html/rfc6143#section-7.2)
- [Maintained RFB protocol specification](https://github.com/rfbproto/rfbproto/blob/master/rfbproto.rst)
