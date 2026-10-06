---
title: "Conditional access: slipping past sign-in policies"
description: "Bypassing Entra conditional-access policies: user-agent and device-platform spoofing, legacy-auth protocols, and location or compliant-device gaps."
keywords:
  - conditional access
  - bypass
  - legacy authentication
  - device compliance
  - policy gap
---

# Conditional access

Conditional-access policies gate sign-in on conditions (app, platform, location, device state), and bypasses come from the gaps between those conditions. If a policy targets only certain apps or platforms, you present as something it does not cover; if it trusts a device-compliance or location signal, you spoof or enter from it.

## Finding and slipping the gaps

```bash
# present a device platform / client the policy does not cover by changing the user-agent
# (e.g. a policy that blocks mobile but not a spoofed desktop or legacy client)
curl -s -A "Mozilla/5.0 (Windows NT 10.0)" https://login.microsoftonline.com/...

# legacy-auth endpoints (Exchange ActiveSync, Autologon) often fall outside modern-auth CA
```

Enumerate policies once you hold a token (`roadrecon` dumps CA policies) and read them for the uncovered app, platform, or client-type.

## Exploitation notes

- Legacy authentication protocols are the classic hole: many policies only apply to modern auth, so basic-auth endpoints bypass them.
- A policy requiring a compliant or hybrid-joined device is defeated by registering or forging such a device (see [device registration](../devices/device-registration.md)).
- Location-based policies trust source IP; a VPN or egress in an allowed range satisfies them.

## Tools

- **ROADtools** (`roadrecon`): dump and analyse CA policies.
- **AADInternals** / **TokenTactics**: authenticate through alternate endpoints.

## References

- [dirkjanm.io: conditional access research](https://dirkjanm.io/)
- [ROADtools](https://github.com/dirkjanm/ROADtools)
- [HackTricks Cloud: conditional access](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
