---
title: "Authentication: attacking VPN access control"
order: 1
description: "VPNs authenticate with pre-shared keys, passwords, certificates, tokens, or combinations of these controls. Attack paths depend on the actual protocol and policy: for example, offline IKE PSK cracking requires an exchange that exposes suitable material, and a valid factor may still be subject to MFA, posture, and authorization checks."
keywords:
  - vpn authentication
  - pre-shared key
  - password
  - certificate
  - mfa
---

# Authentication

A VPN can combine pre-shared keys, passwords, certificates, tokens, device posture, and MFA before authorizing network access. Each factor has distinct attack conditions. For example, a weak IKE PSK can be tested offline when IKEv1 Aggressive Mode exposes the required exchange material, but that workflow does not apply generically to every IPsec or L2TP/IPsec deployment. Passwords may be sprayed or stuffed against product-specific portals or IKE XAUTH, while certificates can be stolen or affected by validation flaws. Recovering one valid factor does not necessarily establish a tunnel: remaining authentication factors, client posture, group policy, and route authorization still apply.

```bash
ike-scan -A -M <target>                         # aggressive mode leaks PSK material (see IKE)
# SSL-VPN login paths and request formats are product-specific; capture and reproduce
# the authorized test portal's actual request rather than using a generic NetExec command
```

## Subtopics

- **[Weak pre-shared keys](weak-pre-shared-keys.md)**: cracking default and weak PSKs.
- **[Password brute force](password-brute-force.md)**: spraying and stuffing the portal.
- **[Certificate abuse](certificate-abuse.md)**: stolen certificates and validation bypass.

## References

- [HackTricks: IPsec/IKE auth](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
- [CISA: VPN credential attacks](https://www.cisa.gov/news-events/cybersecurity-advisories)
