---
title: "Key exchange downgrade: forcing weak SSH key-exchange"
description: "SSH key exchange establishes the session keys, and a server offering weak methods, small Diffie-Hellman groups (group1/1024-bit), or legacy exchanges, can be negotiated down to them. A positioned attacker who can influence the negotiation or who records the handshake targets these weaker exchanges to attack the resulting session keys."
keywords:
  - key exchange
  - diffie-hellman
  - group1
  - downgrade
  - kex
---

# Key exchange downgrade

The SSH key exchange (KEX) derives the symmetric session keys and authenticates the server's host key. Its strength depends on the method negotiated: modern curve-based exchanges are strong, but servers that still offer `diffie-hellman-group1-sha1` (a fixed 1024-bit group) or other legacy KEX present a weaker target. Because both sides pick from offered lists, a server advertising weak groups can end up using them, and a positioned attacker who influences the negotiated algorithm set, or who simply records a handshake that used a weak group, attacks the key agreement to recover or weaken the session keys.

```bash
# which KEX methods does the server offer?
nmap -p22 --script ssh2-enum-algos <target> | grep -A15 kex_algorithms
# force a weak KEX to confirm the server accepts it
ssh -o KexAlgorithms=diffie-hellman-group1-sha1 user@<target>
ssh -o KexAlgorithms=diffie-hellman-group-exchange-sha1 user@<target>
```

## Exploitation notes

- The signal is the server's offered `kex_algorithms`: presence of `group1` (1024-bit), SHA-1-based exchanges, or the GEX with small group sizes marks a weak negotiation that can be forced.
- A 1024-bit fixed group is within reach of well-resourced precomputation attacks against the shared parameters; the practical impact is enabling decryption of a captured session that used it.
- This pairs with a capture or on-path position: forcing or observing a weak KEX is the groundwork for attacking the session, rather than a standalone break.
- The broader weakness set (ciphers, MACs) compounds this; a server offering weak KEX usually offers weak ciphers and MACs too.

## References

- [RFC 4253: key exchange](https://datatracker.ietf.org/doc/html/rfc4253)
- [Logjam and weak DH groups](https://weakdh.org/)
