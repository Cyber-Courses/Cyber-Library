---
title: "Weak cryptography: VNC transport and security-type weaknesses"
description: "Standard VNC does not encrypt the session, so screen contents, keystrokes, and the authentication challenge-response cross the network in cleartext. The RFB security-type negotiation can also be downgraded to a weaker or no-auth type, and the encryption some implementations add is often weak or optional, leaving captured VNC traffic and credentials recoverable."
keywords:
  - vnc encryption
  - cleartext
  - rfb security type
  - downgrade
  - weak crypto
---

# Weak cryptography

Base VNC has no transport encryption: the RFB session, framebuffer updates (the screen), client input (keystrokes and mouse), and even the authentication challenge-response travel in cleartext. A positioned attacker therefore reads the desktop and captures the authentication material. On top of that, the security type is negotiated and can be downgraded to a weaker or no-authentication option, and the encryption that some implementations bolt on (vendor schemes, VeNCrypt/TLS) is often weak, optional, or not verified, so it frequently fails to protect the session in practice.

```bash
# confirm the session is cleartext (no VeNCrypt/TLS security type)
nmap -p5900 --script vnc-info <target>         # security types; absence of VeNCrypt(19) => cleartext
tcpdump -i eth0 -w vnc.pcap port 5900          # capture for analysis
```

## Subtopics

- **[Cleartext transmission](cleartext-transmission.md)**: reading the unencrypted desktop and credentials.
- **[Weak encryption](weak-encryption.md)**: weak or optional encryption modes.
- **[Security type downgrade](security-type-downgrade.md)**: forcing a weaker security type.

## References

- [RFC 6143: security types](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
