---
title: "Service detection: identifying remote-desktop software on hosts and networks"
description: "Endpoint artifacts provide the strongest evidence of installed remote-desktop software. DNS, proxy, firewall, and process-aware connection telemetry can identify outbound relay use, but scanning common destination ports does not generally detect these clients."
keywords:
  - service detection
  - port 5938
  - port 6568
  - relay
  - client artefacts
---

# Service detection

Identifying which remote-desktop product is present comes from two angles. On a host, processes, services, installed files, package inventory, registry entries, and application-data directories name the product precisely. On the network, DNS, proxy, firewall, and process-aware connection telemetry can reveal outbound communication with vendor infrastructure. TeamViewer normally requires no inbound port and initiates outbound connections on 5938, 443, or 80. AnyDesk documents outbound TCP 80, 443, and 6568, plus UDP 50001–50003 for local discovery. Because several products share common destinations such as TCP 443, port numbers alone are not conclusive.

```bash
# host artefacts (Windows)
tasklist | findstr /i 'teamviewer anydesk splashtop logmein'
dir "%APPDATA%\AnyDesk" "%ProgramData%\TeamViewer" 2>nul
# network telemetry: correlate vendor domains/destinations with the originating process
netstat -ano | findstr /r ":5938 :6568 :443 :80"
```

## Exploitation notes

- Inbound scans usually do not identify relay-based clients. An explicitly enabled LAN-listening mode is a separate configuration that must be verified before treating an open port as a product fingerprint.
- Application-data and registry locations expose configuration and identifiers; do not assume they contain a recoverable plaintext unattended password.
- Multiple tools are often installed (a managed one plus a user-installed one); enumerate all, as a forgotten or shadow install may be the weakest.
- The identified product plus its [version](version-detection.md) selects the authentication attack and the applicable [product exploit](../known-product-exploits/index.md).

## References

- [TeamViewer ports](https://www.teamviewer.com/en-us/global/support/knowledge-base/teamviewer-remote/troubleshooting/ports-used-by-teamviewer/)
- [AnyDesk firewall requirements](https://support.anydesk.com/firewall)
- [HackTricks](https://book.hacktricks.xyz/)
