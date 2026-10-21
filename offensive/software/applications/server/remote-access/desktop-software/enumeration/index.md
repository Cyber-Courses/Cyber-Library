---
title: "Enumeration: detecting remote-desktop software and version"
description: "Remote-desktop tools are detected reliably through endpoint inventory and process, file, service, registry, DNS, proxy, or outbound-connection telemetry. Relay destination ports alone are not sufficient to identify an installed product."
keywords:
  - service detection
  - version detection
  - port 5938
  - port 6568
  - fingerprint
---

# Enumeration

Enumerating these tools identifies which product is present and its version, which drives configuration review and advisory matching. Endpoint evidence such as processes, services, installed files, package inventory, registry entries, and application data is authoritative. Network evidence comes from outbound connections, DNS, proxy, and firewall logs for vendor relay infrastructure. TeamViewer normally connects outward on TCP/UDP 5938 with fallbacks to TCP 443 or 80; AnyDesk uses outbound TCP 80, 443, or 6568 and UDP 50001–50003 for local discovery. These destination ports are not general inbound service fingerprints.

```bash
# on a host: identify the client and version from processes/installed files
tasklist | findstr /i 'teamviewer anydesk splashtop'    # Windows
# from network telemetry, correlate destinations and process identity rather than scanning ports
```

## Subtopics

- **[Service detection](service-detection.md)**: identifying the product by port and traffic.
- **[Version detection](version-detection.md)**: determining the client version.

## References

- [TeamViewer ports](https://www.teamviewer.com/en-us/global/support/knowledge-base/teamviewer-remote/troubleshooting/ports-used-by-teamviewer/)
- [AnyDesk firewall requirements](https://support.anydesk.com/firewall)
