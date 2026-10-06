---
title: "Desktop software: attacking third-party remote-desktop tools"
order: 1
description: "Third-party remote-desktop products, TeamViewer, AnyDesk, Chrome Remote Desktop, Splashtop, and similar, provide remote control outside the OS's native RDP/VNC. The surface is their ID-and-password or unattended-access model, weak and default passwords, internet and relay exposure, insecure defaults, and product-specific authentication-bypass and code-execution flaws."
keywords:
  - teamviewer
  - anydesk
  - remote desktop software
  - unattended access
  - third-party
---

# Desktop software

Alongside the OS-native remote-access protocols, organizations and individuals run third-party remote-desktop products, TeamViewer, AnyDesk, Chrome Remote Desktop, Splashtop, LogMeIn, and others, that provide remote control through the vendor's own client, often via a cloud relay that works through NAT and firewalls. That design shifts the attack surface: access is keyed on a device ID plus a password (or unattended-access password), the clients listen locally and connect out to relays, and the products have their own authentication and code-execution vulnerabilities. The surface is the ID/password and unattended-access model, weak and default passwords, internet and relay exposure, insecure default settings, and named product exploits.

```bash
# detect common clients by their characteristic local/relay ports
nmap -p5938,7070,443 -sV <target>              # TeamViewer 5938, AnyDesk 7070, relays on 443
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: detecting the product and version.
- **[Authentication](authentication/index.md)**: ID-based access, unattended access, and weak passwords.
- **[Exposure](exposure/index.md)**: internet exposure and insecure defaults.
- **[Known product exploits](known-product-exploits/index.md)**: TeamViewer and AnyDesk vulnerabilities.

## References

- [TeamViewer security](https://www.teamviewer.com/en/trust-center/security/)
- [AnyDesk security](https://anydesk.com/en/security)
- [HackTricks: remote management software](https://book.hacktricks.xyz/)
