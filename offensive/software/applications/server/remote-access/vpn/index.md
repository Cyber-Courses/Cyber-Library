---
title: "VPN: attacking remote-access VPN gateways and clients"
order: 7
description: "Remote-access VPNs terminate on internet-facing gateways and authenticate remote users into the internal network, making them a top initial-access target. The surface is authentication (PSKs, passwords, certificates), crypto negotiation and downgrade, service and protocol enumeration, protocol-specific flaws (IPsec/IKE, OpenVPN, PPTP), traffic-handling leaks, and the heavily-exploited SSL-VPN appliance vulnerabilities."
keywords:
  - vpn
  - ssl-vpn
  - ipsec
  - ike
  - remote access
---

# VPN

A remote-access VPN gateway sits on the internet and authenticates remote users into the internal network, so compromising one is a direct route from outside to inside, which is why VPN appliances are among the most exploited initial-access targets. The surface spans authentication (pre-shared keys, passwords, certificates, and the MFA often bolted on), crypto negotiation and downgrade, enumeration of the service and its protocols, protocol-specific weaknesses (IPsec/IKE aggressive mode, OpenVPN and PPTP flaws), traffic-handling leaks (DNS, routes, split tunneling), and, most consequentially, the pre-authentication vulnerabilities in the major SSL-VPN appliances.

```bash
# identify VPN services
nmap -sU -p500,4500 <target>                   # IKE/IPsec
nmap -p1723,443,1194 -sV <target>              # PPTP, SSL-VPN/portal, OpenVPN
ike-scan <target>                              # IKE handshake fingerprint
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: protocol and version detection.
- **[Authentication](authentication/index.md)**: PSKs, passwords, and certificates.
- **[Weak cryptography](weak-cryptography/index.md)**: cipher and key-exchange weaknesses.
- **[Protocols](protocols/index.md)**: IPsec/IKE, OpenVPN, PPTP, WireGuard, L2TP/SSTP.
- **[Traffic handling](traffic-handling/index.md)**: DNS, route, and split-tunnel leaks.
- **[SSL-VPN appliances](ssl-vpn-appliances/index.md)**: the pre-auth appliance exploits.

## References

- [NIST SP 800-77 (IPsec VPNs)](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [CISA: VPN exploitation advisories](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [HackTricks: IPsec/IKE VPN](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
