---
title: "Banner grabbing: the VNC RFB protocol version"
description: "A VNC server opens the RFB handshake by sending its protocol version as a fixed 12-byte string, for example RFB 003.008, before any authentication. That version, and the security types the server then offers, identify the protocol generation and the authentication the server will accept, directing the choice of attack."
keywords:
  - rfb version
  - vnc banner
  - handshake
  - security types
  - 5900
---

# Banner grabbing

The RFB protocol begins with the server sending a 12-byte version string such as `RFB 003.008\n`, immediately on connection and before any authentication. Reading it identifies the protocol generation (3.3, 3.7, 3.8), which matters because older versions negotiate security differently (in 3.3 the server dictates the single security type; 3.7+ offer a list the client chooses from, which enables downgrade). After the version, the server presents the security types it accepts, the first real indication of whether authentication is required.

```bash
nc <target> 5900 | head -c 12                  # "RFB 003.008"
nmap -p5900 --script vnc-info <target>         # version + security type list
# the security-type byte(s) that follow: 1 = None, 2 = VNC auth (weak), 16/18 = Tight/VeNCrypt
```

## Exploitation notes

- The version string is pre-auth and free; `003.003` versus `003.007`/`003.008` tells you whether the client can choose the security type (enabling [security type downgrade](../weak-cryptography/security-type-downgrade.md)) or the server dictates it.
- The security types offered right after are the decisive fact: security type 1 is "None" ([no authentication](../authentication/no-authentication.md)), type 2 is the weak VNC DES password, and vendor types (Tight, VeNCrypt) point at implementation-specific handling.
- Combine the version with [implementation detection](implementation-detection.md) to map to known bypasses (RealVNC/TightVNC).
- Displays beyond the first sit on 5901, 5902, etc.; scan the range, as multiple VNC servers on one host are common.

## References

- [RFC 6143: handshake](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
