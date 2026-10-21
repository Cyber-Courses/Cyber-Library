---
title: "Password brute force: attacking the VNC password scheme"
description: "Classic VNC Authentication uses a DES challenge-response and, in compatible implementations, only the first eight password bytes. Captured exchanges permit offline candidate testing, while online guessing depends on server controls; recovery cost still depends on password distribution and cracking resources."
keywords:
  - vnc brute force
  - des challenge
  - offline cracking
  - eight character
  - rfb auth
---

# Password brute force

VNC Authentication (security type 2) is a challenge-response: the server sends a 16-byte challenge and the client returns a DES-based response derived from the password. In classic compatible implementations, only the first eight password bytes are significant. Online guessing submits candidates to the server and is subject to its delay, rejection, and logging behavior. A captured challenge-response pair permits candidates to be tested offline, avoiding further server interaction, but it does not guarantee quick recovery: cost depends on password choice, candidate strategy, character set, and available hardware.

```bash
# online brute force
nmap -p5900 --script vnc-brute --script-args passdb=rockyou.txt <target>
hydra -P rockyou.txt vnc://<target>
# offline: capture the challenge + response (sniff/MITM), then crack
#   VNC auth is a known DES-challenge format; feed challenge+response to a cracker
#   prioritize evidence-based candidate lists; exhaustive recovery may remain costly
```

## Exploitation notes

- In compatible implementations, bytes after the eighth do not strengthen VNC Authentication, but an eight-byte maximum does not make every possible password easy to recover.
- Offline candidate testing avoids server-side throttling and additional authentication logs; recovery speed still depends on the candidate space. See [cleartext transmission](../weak-cryptography/cleartext-transmission.md) for transport context.
- Online brute force works where you cannot capture, but VNC servers may limit or delay attempts; keep lists short given the 8-char cap.
- Test recovered credentials only where reuse is in scope; reuse across endpoints is an operational possibility, not a protocol property.

## Tools

- [nmap vnc-brute](https://nmap.org/nsedoc/scripts/vnc-brute.html)
- [hydra](https://github.com/vanhauser-thc/thc-hydra)

## References

- [RFC 6143: VNC authentication](https://datatracker.ietf.org/doc/html/rfc6143#section-7.2.2)
- [HackTricks: VNC brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
