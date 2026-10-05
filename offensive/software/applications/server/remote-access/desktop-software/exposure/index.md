---
title: "Exposure: internet-facing and insecurely-configured remote-desktop tools"
description: "Remote-desktop tools reach machines through vendor relays, so they work from anywhere, which combined with weak or default configuration exposes those machines broadly. The surface is internet-reachable installations discoverable through enumeration, and insecure defaults, enabled unattended access, weak password policy, no allowlist, that leave a client open to connection."
keywords:
  - exposure
  - internet-facing
  - insecure defaults
  - unattended access
  - relay
---

# Exposure

By design these tools connect through the vendor's cloud relay, so any installed client is reachable from anywhere the attacker can reach that relay, which effectively means the internet, without the machine itself being directly internet-facing. That reach, combined with weak configuration, is the exposure. The two aspects are internet-reachable installations (any running client with a known ID and a weak/leaked password is connectable from outside) and insecure defaults, unattended access enabled, weak or no password policy, no connection allowlist or approval requirement, that leave the client open. The relay model means "internal only" is not a real boundary for these tools.

## Subtopics

- **[Internet exposure](internet-exposure.md)**: reachability through the vendor relay.
- **[Insecure defaults](insecure-defaults.md)**: configurations that leave a client open.

## References

- [TeamViewer / AnyDesk connectivity model](https://www.teamviewer.com/en/)
- [HackTricks](https://book.hacktricks.xyz/)
