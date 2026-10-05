---
title: "Weak cryptography: VPN cipher and key-exchange weaknesses"
description: "VPN gateways that negotiate weak ciphers or key-exchange parameters, legacy ciphers, small Diffie-Hellman groups, SHA-1, let a positioned attacker select or downgrade to breakable crypto. This enables decryption of captured tunnel traffic or attacks on the key agreement, undermining the confidentiality the VPN exists to provide."
keywords:
  - vpn crypto
  - cipher
  - key exchange
  - diffie-hellman
  - downgrade
---

# Weak cryptography

A VPN's whole purpose is confidentiality, so its negotiated cryptography is the target. Gateways that still offer weak options, legacy ciphers (DES/3DES, RC4), small Diffie-Hellman groups (768/1024-bit), or SHA-1 integrity, give a positioned attacker a breakable session: the negotiation can land on, or be downgraded to, the weak option, after which captured tunnel traffic is decryptable or the key agreement is attackable. This matters most for IPsec/IKE (where transform sets are explicitly offered) and for TLS-based VPNs with weak cipher suites, and it compounds the authentication and protocol weaknesses.

```bash
# IKE transform sets offered (weak DH groups, DES/3DES, SHA-1?)
ike-scan -M <target>
# TLS-based VPN (SSL-VPN/SSTP/OpenVPN-TLS) cipher grading
nmap -p443 --script ssl-enum-ciphers <target>
```

## Subtopics

- **[Weak ciphers](weak-ciphers.md)**: legacy and broken cipher negotiation.
- **[Key exchange downgrade](key-exchange-downgrade.md)**: small or weak key-exchange groups.

## References

- [NIST SP 800-77: VPN crypto](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [Logjam / weak DH](https://weakdh.org/)
