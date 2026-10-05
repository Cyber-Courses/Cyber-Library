---
title: "Cleartext transmission: reading the unencrypted VNC session"
description: "Standard VNC sends the framebuffer, input events, and the authentication challenge-response without encryption. A positioned attacker captures the traffic to reconstruct the remote screen, recover keystrokes including typed passwords, and extract the challenge-response pair for offline cracking of the VNC password."
keywords:
  - cleartext
  - framebuffer capture
  - keystroke capture
  - challenge response
  - sniffing
---

# Cleartext transmission

Without an encrypting security type (VeNCrypt/TLS), a VNC session is entirely in the clear on the wire. For an attacker with a capture position this yields three things: the framebuffer updates, which reconstruct the remote desktop image the user sees; the client input events, which include every keystroke, so passwords typed into the session are captured; and the authentication handshake, whose challenge and response can be lifted for offline cracking of the short VNC password. A single captured session can thus give both the content and the credential.

```bash
# capture VNC traffic from an on-path position
tcpdump -i eth0 -w vnc.pcap port 5900
# reconstruct the screen from framebuffer updates, and read input events, in analysis
#   (tools and Wireshark dissectors parse RFB framebuffer/keyevent messages)
# extract the auth challenge+response for offline cracking (see password brute force)
```

## Exploitation notes

- Three distinct wins from one capture: screen reconstruction (what the user is doing), keystroke recovery (passwords and commands they type), and the challenge-response pair for [offline password cracking](../authentication/password-brute-force.md).
- Capturing keystrokes often yields credentials to other systems that the user types into the VNC session, beyond the VNC password itself.
- The attack needs an on-path position (ARP/DNS/route) or a tap; VNC over a flat LAN is readily intercepted.
- The defense is tunnelling VNC (SSH/VPN) or using VeNCrypt/TLS; the absence of an encrypting security type in enumeration confirms the session is exposed.

## References

- [RFC 6143: no-encryption default](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC sniffing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
