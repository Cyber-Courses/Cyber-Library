---
title: "Banner grabbing: the VNC RFB protocol version"
description: "An RFB server begins with a 12-byte protocol-version string. After the client replies with a compatible version, an RFB 3.7 or later server lists its supported security types; use a protocol-aware client such as Nmap's vnc-info to complete this exchange."
keywords:
  - rfb version
  - vnc banner
  - handshake
  - security types
  - 5900
---

# Banner grabbing

The RFB protocol begins with the server sending a 12-byte version string such as `RFB 003.008\n`, before authentication. The client must reply with the compatible protocol version it intends to use. Under RFB 3.3 the server then selects a single security type; under 3.7 and later the server sends a list and the client selects a supported type. The list identifies available authentication and transport mechanisms. A downgrade opportunity exists only when an on-path attacker can alter negotiation and both endpoints support a weaker type that the server genuinely offered.

```bash
nc <target> 5900 | head -c 12                  # "RFB 003.008"
nmap -p5900 --script vnc-info <target>         # version + security type list
# selected registered values: 1=None, 2=VNC Authentication, 16=Tight,
# 18=TLS, 19=VeNCrypt, 20=SASL
```

## Exploitation notes

- The version string is pre-auth; `003.003` versus `003.007`/`003.008` determines whether the server dictates one type or the client chooses from the server's list.
- The security types offered right after are the decisive fact: security type 1 is "None" ([no authentication](../authentication/no-authentication.md)), type 2 is the weak VNC DES password, and vendor types (Tight, VeNCrypt) point at implementation-specific handling.
- Combine protocol evidence with [implementation detection](implementation-detection.md), then match only named advisories to confirmed products and versions.
- Displays beyond the first sit on 5901, 5902, etc.; scan the range, as multiple VNC servers on one host are common.

## References

- [RFC 6143: protocol version](https://datatracker.ietf.org/doc/html/rfc6143#section-7.1.1)
- [RFC 6143: security negotiation](https://datatracker.ietf.org/doc/html/rfc6143#section-7.1.2)
- [Maintained RFB security-type registry](https://github.com/rfbproto/rfbproto/blob/master/rfbproto.rst)
