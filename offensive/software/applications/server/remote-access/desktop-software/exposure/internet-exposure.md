---
title: "External reachability: remote-desktop vendor relays"
description: "Vendor relays can provide an external connection path without a conventional inbound port. Actual reachability and access depend on client state, outbound network access, incoming-session policy, approval or unattended-access configuration, authentication, allowlists, and other controls."
keywords:
  - internet exposure
  - relay
  - reachability
  - device id
  - connectable
---

# External reachability

Relay-based remote-desktop products commonly establish outbound connections to vendor infrastructure, so a session may be brokered without a conventional inbound port on the endpoint. That architecture can create an external access path through NAT, but it does not make every running client remotely controllable. The client must reach the relay and permit the requested incoming session, and the operator must satisfy the configured approval, authentication, allowlist, account, device-trust, and permission requirements. Egress filtering, DNS or proxy policy, application control, conditional access, and self-hosted or on-premises designs can further restrict the path.

```bash
# review endpoint and network evidence together
# - is the client or service running?
# - can it reach the vendor relay through DNS, proxy, and firewall policy?
# - which incoming-session and authorization controls apply?
```

## Exploitation notes

- Network segmentation and egress controls remain relevant because they can limit which endpoints communicate with public relay infrastructure.
- A device ID identifies a destination but does not authenticate a connection; review the controls described in [ID-based access](../authentication/id-based-access.md).
- Interactive sessions require user acceptance under applicable policy, while unattended sessions require their configured authentication and authorization controls.
- Pair reachability analysis with [configuration review](insecure-defaults.md) rather than inferring access from installation or relay traffic alone.

## References

- [TeamViewer ports and URLs](https://www.teamviewer.com/en-us/global/support/knowledge-base/teamviewer-remote/troubleshooting/ports-used-by-teamviewer/)
- [TeamViewer restrict access](https://community.teamviewer.com/English/kb/articles/108687-restrict-access)
- [AnyDesk firewall requirements](https://support.anydesk.com/firewall)
- [AnyDesk interactive access](https://support.anydesk.com/docs/interactive-access)
