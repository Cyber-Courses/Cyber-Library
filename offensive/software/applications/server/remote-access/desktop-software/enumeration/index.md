---
title: "Enumeration: detecting remote-desktop software and version"
description: "Remote-desktop tools are detected by their characteristic local listening ports and relay traffic, TeamViewer on 5938, AnyDesk on 7070, and others, and by client artefacts on a host. Identifying the product and its version selects the authentication attack and maps the client to its known product exploits."
keywords:
  - service detection
  - version detection
  - port 5938
  - port 7070
  - fingerprint
---

# Enumeration

Enumerating these tools identifies which product is present and its version, which drives both the authentication attack and the product-exploit choice. Detection is by the characteristic ports the clients use, TeamViewer on TCP 5938 (falling back to 443/80), AnyDesk on 7070, Splashtop and others on their own, and by the vendor relay traffic, plus client artefacts (processes, installed files, registry) on a host you can inspect. Version detection then matters because these products patch frequently and many attacks (brute-force feasibility, specific exploits) are version-dependent.

```bash
nmap -p5938,7070,6568,443 -sV <target>         # TeamViewer, AnyDesk, Splashtop, relay
# on a host: identify the client and version from processes/installed files
tasklist | findstr /i 'teamviewer anydesk splashtop'    # Windows
```

## Subtopics

- **[Service detection](service-detection.md)**: identifying the product by port and traffic.
- **[Version detection](version-detection.md)**: determining the client version.

## References

- [TeamViewer ports](https://www.teamviewer.com/en/)
- [AnyDesk ports](https://anydesk.com/en/)
