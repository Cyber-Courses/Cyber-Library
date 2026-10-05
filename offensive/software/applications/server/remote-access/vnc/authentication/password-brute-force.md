---
title: "Password brute force: attacking the VNC password scheme"
description: "The classic VNC authentication is a DES challenge-response over a password truncated to eight characters. It is brute-forced online against the server, and, more powerfully, cracked offline from a single captured challenge-response pair, because the short password space and known algorithm make offline recovery fast."
keywords:
  - vnc brute force
  - des challenge
  - offline cracking
  - eight character
  - rfb auth
---

# Password brute force

VNC's standard authentication (security type 2) is a challenge-response: the server sends a 16-byte challenge, the client encrypts it with DES keyed by the password (truncated to eight characters, with a quirk that reverses the bit order of each key byte), and returns the result. Two attacks follow. Online brute force tries passwords against the server directly. More powerfully, capturing one challenge-response pair (by sniffing or MITM) allows offline cracking: the password space is tiny (eight characters, and the algorithm is known), so a wordlist or brute force recovers the password quickly without touching the server again.

```bash
# online brute force
nmap -p5900 --script vnc-brute --script-args passdb=rockyou.txt <target>
hydra -P rockyou.txt vnc://<target>
# offline: capture the challenge + response (sniff/MITM), then crack
#   VNC auth is a known DES-challenge format; feed challenge+response to a cracker
#   (the 8-char cap and reversed-bit key make the keyspace small and fast to search)
```

## Exploitation notes

- The eight-character truncation is the decisive weakness: only the first eight characters are keyed, so the effective space is small and wordlists are highly effective; longer passwords add nothing beyond eight bytes.
- Offline cracking from a captured challenge-response is quiet and fast, and avoids online lockout/logging; a single sniffed authentication (the traffic is otherwise cleartext) yields the pair, see [cleartext transmission](../weak-cryptography/cleartext-transmission.md).
- Online brute force works where you cannot capture, but VNC servers may limit or delay attempts; keep lists short given the 8-char cap.
- A cracked password is reusable across VNC endpoints and sometimes other services; the recovered desktop access is interactive control of the target.

## Tools

- [nmap vnc-brute](https://nmap.org/nsedoc/scripts/vnc-brute.html)
- [hydra](https://github.com/vanhauser-thc/thc-hydra)

## References

- [RFC 6143: VNC authentication](https://datatracker.ietf.org/doc/html/rfc6143#section-7.2.2)
- [HackTricks: VNC brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
