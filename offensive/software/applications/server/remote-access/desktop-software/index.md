---
title: "Desktop software: attacking third-party remote-desktop tools"
order: 1
description: "Third-party remote-desktop products provide remote control outside native RDP or VNC. Review endpoint inventory, relay reachability, product-specific authentication and authorization controls, client configuration, and precisely scoped vendor advisories."
keywords:
  - teamviewer
  - anydesk
  - remote desktop software
  - unattended access
  - third-party
---

# Desktop software

Alongside the OS-native remote-access protocols, organizations and individuals run third-party remote-desktop products, TeamViewer, AnyDesk, Chrome Remote Desktop, Splashtop, LogMeIn, and others, that provide remote control through the vendor's own client, often via a cloud relay that works through NAT and firewalls. That design shifts the attack surface: access is keyed on a device ID plus a password, account assignment, or an unattended-access policy; clients normally initiate outbound connections to vendor infrastructure; and product-specific vulnerabilities affect particular versions. The surface includes access-control configuration, weak or exposed credentials, relay reachability, and named product advisories.

```bash
# endpoint inventory is authoritative; relay traffic is normally outbound
tasklist | findstr /i 'teamviewer anydesk splashtop logmein'
netstat -ano | findstr /r ":5938 :6568 :443 :80"
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: detecting the product and version.
- **[Authentication](authentication/index.md)**: ID-based access, unattended access, and weak passwords.
- **[Exposure](exposure/index.md)**: external reachability and incoming-session configuration.
- **[Known product issues](known-product-exploits/index.md)**: version-specific advisories and documented security incidents.

## References

- [TeamViewer security](https://www.teamviewer.com/en/trust-center/security/)
- [AnyDesk security](https://anydesk.com/en/security)
- [HackTricks: remote management software](https://book.hacktricks.xyz/)
