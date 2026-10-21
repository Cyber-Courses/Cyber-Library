---
title: "Weak cryptography: VPN cipher and key-exchange weaknesses"
order: 2
description: "VPN gateways that negotiate weak ciphers or key-exchange parameters, legacy ciphers, small Diffie-Hellman groups, or SHA-1 can expose compatible clients to obsolete cryptography. This weakens the tunnel and may enable protocol-specific attacks under the conditions required by the negotiated algorithm."
keywords:
  - vpn crypto
  - cipher
  - key exchange
  - diffie-hellman
  - downgrade
---

# Weak cryptography

A VPN's whole purpose is confidentiality, so its negotiated cryptography matters. Gateways that still offer legacy ciphers (DES/3DES or RC4), small Diffie-Hellman groups (768/1024-bit), or SHA-1 integrity permit compatible clients to establish tunnels with obsolete protection. Offering a weak option is evidence of insecure configuration, but it does not by itself prove that an unauthenticated attacker can force a downgrade: IKE and TLS authenticate their completed negotiations. Determine which algorithms a real client negotiates, then assess the algorithm-specific attack conditions, such as the traffic volume required by a 64-bit block-cipher attack or the resources required to attack a finite-field DH group.

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
