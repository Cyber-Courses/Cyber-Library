---
title: "Exposure: externally reachable and insecurely configured remote-desktop tools"
description: "Many remote-desktop products use outbound connections to vendor relay infrastructure. Whether an endpoint accepts an external session still depends on service state, egress reachability, incoming-session policy, interactive approval, unattended-access configuration, authentication, allowlists, and other controls."
keywords:
  - exposure
  - internet-facing
  - insecure defaults
  - unattended access
  - relay
---

# Exposure

Many remote-desktop products initiate outbound connections to vendor infrastructure, allowing sessions without exposing a conventional inbound listening port. This creates an externally mediated access path when the client is running, can reach the service, and is configured to accept the requested connection. Interactive approval, unattended-access state, passwords or account assignment, multifactor authentication, allowlists, permission profiles, conditional-access policy, and network egress controls can all affect whether the path is usable. Review relay reachability and client authorization together rather than treating installation alone as internet exposure.

## Subtopics

- **[External reachability](internet-exposure.md)**: relay-mediated connectivity and its control points.
- **[Insecure configuration](insecure-defaults.md)**: settings that weaken incoming-session authorization.

## References

- [TeamViewer ports and URLs](https://www.teamviewer.com/en-us/global/support/knowledge-base/teamviewer-remote/troubleshooting/ports-used-by-teamviewer/)
- [AnyDesk firewall requirements](https://support.anydesk.com/firewall)
