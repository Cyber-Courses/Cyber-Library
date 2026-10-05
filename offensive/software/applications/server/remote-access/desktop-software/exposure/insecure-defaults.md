---
title: "Insecure defaults: remote-desktop configurations that leave a client open"
description: "Remote-desktop clients are frequently left in insecure configurations: unattended access enabled with a weak password, no connection allowlist or approval prompt, weak password policy, and disabled additional authentication. These defaults, combined with the tool's internet reach, leave the machine open to connection by anyone who obtains the ID and password."
keywords:
  - insecure defaults
  - unattended access
  - allowlist
  - approval prompt
  - configuration
---

# Insecure defaults

The configuration of these clients determines how open a reachable machine is, and the insecure settings are common. Unattended access enabled with a weak, reused, or never-changed password gives standing connectability. The absence of a connection allowlist (restricting which IDs may connect) or an approval prompt means any client with the ID and password connects unchallenged. Weak password policy allows short/guessable passwords, and additional protections (two-factor, device approval) are often not enabled. Combined with the tool's relay-based [internet reach](internet-exposure.md), these defaults leave the machine open to connection by anyone who obtains the ID and password.

```bash
# on a host, review the client configuration for the open settings
#   unattended access enabled? allowlist configured? approval required? 2FA on?
#   (locations: TeamViewer/AnyDesk registry and app-data config)
reg query 'HKLM\SOFTWARE\WOW6432Node\TeamViewer' 2>nul      # inspect settings
# weak unattended password + no allowlist + no 2FA = open to anyone with ID+password
```

## Exploitation notes

- The riskiest default is unattended access with a weak/reused password and no allowlist: it is standing, unchallenged remote control for whoever has the credential, see [unattended access](../authentication/unattended-access.md).
- A missing connection allowlist and approval prompt mean no second gate beyond the password; where 2FA/device approval is available but disabled, the password is the only barrier.
- Inspect these settings on any host you reach; a permissively-configured client is both an access path and a persistence mechanism.
- These defaults are what make the tool's [internet exposure](internet-exposure.md) actionable; together they explain the heavy abuse of these products for remote access.

## References

- [TeamViewer hardening/allowlist](https://www.teamviewer.com/en/trust-center/security/)
- [AnyDesk security settings](https://anydesk.com/en/security)
