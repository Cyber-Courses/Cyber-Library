---
title: "Password brute force: spraying and stuffing the VPN portal"
description: "SSL-VPN portals and IKE XAUTH accept corporate username and password, making them prime targets for password spraying and credential stuffing. Because VPN logins use the same directory credentials as everything else, a sprayed or breached password frequently grants a tunnel into the internal network, often where the VPN is the perimeter."
keywords:
  - vpn brute force
  - password spray
  - credential stuffing
  - xauth
  - sslvpn portal
---

# Password brute force

VPN authentication usually validates against the corporate directory, so the SSL-VPN portal and IKE XAUTH are high-value targets for password spraying and credential stuffing: a single working password is a tunnel into the internal network, and because the VPN often is the perimeter, that is deep access. Spraying one common password across many users avoids lockout and finds weak accounts; stuffing replays breached and infostealer-sourced credentials, which hit because VPN logins reuse the same passwords as everything else. MFA, where enforced, is the remaining barrier (and is sometimes bypassable or not applied to all auth paths).

```bash
# SSL-VPN portal spraying (product-specific login endpoint)
# generic HTTP-POST form spray with a lockout-safe single password across users:
hydra -L users.txt -p 'Spring2025!' <target> https-post-form \
  "/remote/logincheck:username=^USER^&credential=^PASS^:<fail-string>"
# IKE XAUTH credential attack (after PSK/main-mode)
# learn the lockout policy and spray below it; stuff breached pairs 1:1
```

## Exploitation notes

- Spray one password across validated users to stay under lockout, and stuff breached/infostealer pairs separately; VPN portals are a top stuffing target because they accept reused corporate credentials.
- Derive usernames and the password policy from other enumeration (the directory, OSINT, portal error differences); match the username format the portal expects (UPN, `DOMAIN\user`, or bare).
- The login endpoint and failure signal are product-specific (FortiGate `/remote/logincheck`, others differ); fingerprint the appliance first, see [banner grabbing](../enumeration/banner-grabbing.md).
- A valid credential without MFA is a full tunnel; where MFA is enforced, note auth paths that may skip it (legacy IKE XAUTH, specific portals) and the [appliance exploits](../ssl-vpn-appliances/index.md) that bypass auth entirely.

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [NetExec](https://github.com/Pennyw0rth/NetExec)

## References

- [CISA: VPN password spraying](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [HackTricks: VPN](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
