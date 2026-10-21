---
title: "Cleartext transmission: reading the unencrypted VNC session"
description: "RFB sessions using None or VNC Authentication send subsequent protocol data without encryption. An on-path capture can expose framebuffer updates, input events, and, for VNC Authentication, the challenge-response exchange; protected security types require separate analysis."
keywords:
  - cleartext
  - framebuffer capture
  - keystroke capture
  - challenge response
  - sniffing
---

# Cleartext transmission

When a session selects `None` or VNC Authentication, subsequent RFB protocol data is unencrypted. An on-path capture can expose framebuffer updates and client input events; the VNC Authentication exchange also exposes the challenge and response for offline candidate testing. Practical reconstruction depends on the encodings, extensions, and completeness of the capture. Do not infer cleartext merely because VeNCrypt is absent: TLS, SASL, RSA-AES, Secure Tunnel, Integrated SSH, and vendor mechanisms may provide other forms of protection.

```bash
# capture VNC traffic from an on-path position
tcpdump -i eth0 -w vnc.pcap port 5900
# reconstruct the screen from framebuffer updates, and read input events, in analysis
#   (tools and Wireshark dissectors parse RFB framebuffer/keyevent messages)
# extract the auth challenge+response for offline cracking (see password brute force)
```

## Exploitation notes

- A complete cleartext capture may permit screen reconstruction and recovery of client input events, subject to the negotiated encodings and extensions.
- VNC Authentication challenge-response data permits [offline candidate testing](../authentication/password-brute-force.md), but does not guarantee password recovery.
- The attack needs an on-path position (ARP/DNS/route) or a tap; VNC over a flat LAN is readily intercepted.
- Confirm the selected security type and its effective protection. TLS, VeNCrypt, SASL, RSA-AES, SSH, VPN, or another authenticated encrypted transport may protect the session.

## References

- [RFC 6143: None](https://datatracker.ietf.org/doc/html/rfc6143#section-7.2.1)
- [RFC 6143: VNC Authentication](https://datatracker.ietf.org/doc/html/rfc6143#section-7.2.2)
- [Maintained RFB security types](https://github.com/rfbproto/rfbproto/blob/master/rfbproto.rst)
