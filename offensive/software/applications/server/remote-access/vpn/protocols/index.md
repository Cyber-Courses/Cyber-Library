---
title: "Protocols: attacking specific VPN protocols"
order: 4
description: "VPN protocols differ sharply in their weaknesses: IPsec/IKE exposes aggressive-mode PSK disclosure, OpenVPN's security rests on configuration and key handling, PPTP's MS-CHAPv2 is cryptographically broken, WireGuard is sound but leaks through static-key management, and L2TP and SSTP inherit IPsec-PSK and TLS weaknesses respectively. The protocol in use dictates the attack."
keywords:
  - ipsec
  - ike
  - openvpn
  - pptp
  - wireguard
---

# Protocols

VPNs run over different protocols, and each carries its own characteristic weaknesses, so identifying the protocol selects the attack. IPsec with IKE exposes aggressive-mode pre-shared-key hash disclosure and transform weaknesses. OpenVPN is cryptographically solid but its security depends entirely on configuration and key/certificate handling. PPTP is obsolete and its MS-CHAPv2 authentication is effectively broken. WireGuard's protocol is sound, so the attack shifts to its static-key management and identity exposure. And L2TP and SSTP inherit the weaknesses of what wraps them, an IPsec pre-shared key and TLS respectively.

## Subtopics

- **[IPsec IKE](ipsec-ike.md)**: aggressive-mode PSK disclosure and IKE weaknesses.
- **[OpenVPN](openvpn.md)**: configuration, credential, and key-handling attacks.
- **[PPTP](pptp.md)**: the broken MS-CHAPv2 authentication.
- **[WireGuard](wireguard.md)**: static-key management and identity exposure.
- **[L2TP and SSTP](l2tp-and-sstp.md)**: inherited IPsec-PSK and TLS weaknesses.

## References

- [NIST SP 800-77 (VPN protocols)](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [HackTricks: IPsec/IKE](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
