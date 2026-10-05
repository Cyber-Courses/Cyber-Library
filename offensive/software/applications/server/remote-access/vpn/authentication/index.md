---
title: "Authentication: attacking VPN access control"
description: "VPNs authenticate with pre-shared keys, username and password, certificates, or a combination, often with MFA layered on. Each is attacked: weak or default PSKs are cracked, passwords are sprayed and stuffed against the portal, and certificates are stolen or their validation bypassed. Valid VPN authentication grants a tunnel into the internal network."
keywords:
  - vpn authentication
  - pre-shared key
  - password
  - certificate
  - mfa
---

# Authentication

A VPN authenticates remote users before granting a tunnel, and the methods, pre-shared keys, passwords, certificates, and layered MFA, are each attackable. Weak or default pre-shared keys (common in IPsec and L2TP setups) are captured and cracked offline. Passwords are brute-forced, sprayed, and credential-stuffed against the SSL-VPN portal or IKE XAUTH, and VPN portals are a prime target for stuffing because they accept corporate credentials. Certificates are stolen from clients or their validation bypassed. Because success yields a tunnel into the internal network, VPN authentication is a high-value target, and MFA, where present, is the next hurdle (sometimes itself bypassable).

```bash
ike-scan -A -M <target>                         # aggressive mode leaks PSK material (see IKE)
nxc <sslvpn-portal> ...                          # product-specific portal spraying
```

## Subtopics

- **[Weak pre-shared keys](weak-pre-shared-keys.md)**: cracking default and weak PSKs.
- **[Password brute force](password-brute-force.md)**: spraying and stuffing the portal.
- **[Certificate abuse](certificate-abuse.md)**: stolen certificates and validation bypass.

## References

- [HackTricks: IPsec/IKE auth](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
- [CISA: VPN credential attacks](https://www.cisa.gov/news-events/cybersecurity-advisories)
