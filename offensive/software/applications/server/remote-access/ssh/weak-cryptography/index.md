---
title: "Weak cryptography: attacking SSH cipher, MAC, and key-exchange negotiation"
order: 2
description: "SSH negotiates its cipher, MAC, and key-exchange algorithms, and a server that still offers weak ones, legacy ciphers, MD5-based or truncated MACs, small Diffie-Hellman groups, lets a positioned attacker select or downgrade to a breakable session. These weaknesses enable decryption or tampering of traffic an attacker can capture or relay."
keywords:
  - ssh crypto
  - cipher
  - mac
  - key exchange
  - downgrade
---

# Weak cryptography

SSH begins each session by negotiating algorithms for key exchange, encryption, and message authentication from the lists both sides offer. If a server still advertises weak options, the negotiation (or a positioned attacker forcing it) can land on a breakable combination: legacy or broken ciphers, weak or truncated MACs, and small or flawed key-exchange groups. These do not matter against a passive observer of a strong session, but they enable decryption, tampering, or downgrade for an attacker who can capture or sit on the connection, and they are the enabling weakness behind some SSH machine-in-the-middle and traffic attacks.

```bash
# enumerate offered algorithms and grade them
nmap -p22 --script ssh2-enum-algos <target>
ssh -Q cipher; ssh -Q mac; ssh -Q kex          # what your client supports (for comparison)
# connect forcing a weak algorithm to test acceptance
ssh -c aes128-cbc -o MACs=hmac-md5 user@<target>
```

## Subtopics

- **[Key exchange downgrade](key-exchange-downgrade.md)**: forcing weak or small key-exchange groups.
- **[Weak ciphers](weak-ciphers.md)**: legacy and CBC-mode cipher weaknesses.
- **[Weak MAC algorithms](weak-mac-algorithms.md)**: broken or truncated integrity protection.

## References

- [RFC 4253 (SSH transport layer)](https://datatracker.ietf.org/doc/html/rfc4253)
- [nmap ssh2-enum-algos](https://nmap.org/nsedoc/scripts/ssh2-enum-algos.html)
