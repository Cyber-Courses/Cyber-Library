---
title: "Enumeration: fingerprinting a VPN endpoint"
order: 3
description: "VPN enumeration identifies which VPN protocol and product a gateway runs, from the UDP IKE handshake (IPsec), the PPTP control port, or the SSL-VPN web portal, and extracts version and configuration detail. That identification selects the protocol attack and, for appliances, maps the exact build to its pre-authentication exploits."
keywords:
  - vpn enumeration
  - ike-scan
  - protocol detection
  - portal
  - version
---

# Enumeration

VPN enumeration first determines what the gateway is: the UDP IKE handshake on 500/4500 indicates IPsec, TCP 1723 indicates PPTP, a TLS web portal on 443 indicates an SSL-VPN appliance (and which vendor), and 1194 suggests OpenVPN. Then it extracts version and configuration: the IKE transform sets and whether aggressive mode is offered, the SSL-VPN portal's product and build (from the login page, headers, and resources), and TLS details. This drives everything, the protocol determines the attack, and for appliances the exact build maps to its pre-authentication exploit.

```bash
nmap -sU -p500,4500 --script ike-version <target>    # IPsec/IKE
ike-scan -M <target>; ike-scan -A -M <target>        # main vs aggressive mode transforms
nmap -p1723 --script pptp-version <target>           # PPTP
curl -skI https://<target>/                          # SSL-VPN portal product/headers
```

## Subtopics

- **[Protocol detection](protocol-detection.md)**: identifying the VPN protocol and ports.
- **[Banner grabbing](banner-grabbing.md)**: version and product disclosure.

## References

- [ike-scan](https://github.com/royhills/ike-scan)
- [HackTricks: IPsec/IKE enumeration](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
