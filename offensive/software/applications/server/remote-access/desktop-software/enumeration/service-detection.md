---
title: "Service detection: identifying remote-desktop software by port and traffic"
description: "Each remote-desktop product uses characteristic ports and relay endpoints, TeamViewer on 5938 (then 443/80), AnyDesk on 7070, Splashtop on its own ports, so scanning and traffic analysis identify which is running. On a host, processes, installed files, and registry keys confirm the product and point to its stored configuration and credentials."
keywords:
  - service detection
  - port 5938
  - port 7070
  - relay
  - client artefacts
---

# Service detection

Identifying which remote-desktop product is present comes from two angles. On the network, the clients use characteristic ports, TeamViewer prefers TCP 5938 and falls back to 443/80, AnyDesk uses 7070 (and relays over 443), Splashtop and others use their own, so a targeted scan plus observation of connections to the vendor's relay infrastructure reveals the product. On a host you can inspect, the running processes, installed program files, and registry/app-data entries name the product precisely and point to where its configuration and any stored credentials live, which matters for unattended-password recovery.

```bash
# network detection
nmap -p5938,7070,6568,443,80 -sV <target>
# connections to vendor relays indicate the product even when local ports are filtered
# host artefacts (Windows)
tasklist | findstr /i 'teamviewer anydesk splashtop logmein'
dir "%APPDATA%\AnyDesk" "%ProgramData%\TeamViewer" 2>nul
```

## Exploitation notes

- The characteristic ports and relay destinations identify the product even behind NAT, because the client connects outward to the vendor relay; watch for those connections where inbound ports are filtered.
- On a host, the product's app-data/registry location is also where its configuration and unattended credentials are stored, so service detection feeds [unattended-access](../authentication/unattended-access.md) credential recovery.
- Multiple tools are often installed (a managed one plus a user-installed one); enumerate all, as a forgotten or shadow install may be the weakest.
- The identified product plus its [version](version-detection.md) selects the authentication attack and the applicable [product exploit](../known-product-exploits/index.md).

## References

- [TeamViewer / AnyDesk port documentation](https://www.teamviewer.com/en/)
- [HackTricks](https://book.hacktricks.xyz/)
