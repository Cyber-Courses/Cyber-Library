---
title: "Weak ciphers: legacy and broken VPN cipher negotiation"
description: "VPN gateways offering legacy ciphers, single or triple DES, RC4, or export-grade suites, let a positioned attacker negotiate or downgrade to encryption weak enough to decrypt captured tunnel traffic. IPsec exposes this through its offered transform sets and TLS-based VPNs through their cipher suites."
keywords:
  - weak cipher
  - 3des
  - rc4
  - transform set
  - decryption
---

# Weak ciphers

A VPN that still offers weak encryption undermines its own confidentiality. In IPsec the gateway advertises transform sets, and acceptance of DES, 3DES, or weak modes lets a session use breakable encryption. In TLS-based VPNs (SSL-VPN portals, SSTP, OpenVPN-over-TLS) weak cipher suites (RC4, export grade, 64-bit block ciphers vulnerable to birthday attacks) do the same. A positioned attacker who captures the tunnel, or who forces the negotiation toward the weak option, can then decrypt or weaken the traffic the VPN is meant to protect.

```bash
# IPsec: which transforms are offered (look for DES/3DES, weak hashes)
ike-scan -M <target>
# TLS VPN: grade the cipher suites
nmap -p443 --script ssl-enum-ciphers <target>        # flags RC4, 3DES, export, SWEET32-prone
# capture tunnel traffic for offline analysis where a weak suite is in use
```

## Exploitation notes

- The offered transform sets (IPsec) and cipher suites (TLS VPNs) are the evidence; DES/3DES, RC4, and export-grade acceptance mark a gateway whose traffic is attackable with capture.
- 64-bit block ciphers (DES/3DES) are vulnerable to birthday-bound collision attacks on long-lived tunnels (the SWEET32 class), relevant because VPN tunnels carry large volumes over long sessions.
- Weak crypto matters to an attacker who can capture or sit on the tunnel; combine with a capture position, and with [key-exchange downgrade](key-exchange-downgrade.md) which attacks the key agreement itself.
- Even where decryption is impractical, weak-cipher support fingerprints an old gateway likely vulnerable to the [appliance](../ssl-vpn-appliances/index.md) or protocol flaws.

## References

- [SWEET32 (64-bit block cipher attacks)](https://sweet32.info/)
- [nmap ssl-enum-ciphers](https://nmap.org/nsedoc/scripts/ssl-enum-ciphers.html)
