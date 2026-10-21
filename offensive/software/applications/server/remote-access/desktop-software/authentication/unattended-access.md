---
title: "Unattended access: abusing always-on remote-desktop access"
description: "Unattended access permits authorized connections without a user present. Its controls vary by product and can include passwords, account assignment, MFA, allowlists, and managed-device trust; local configuration should not be assumed to expose a reusable password."
keywords:
  - unattended access
  - unattended password
  - persistent access
  - teamviewer
  - anydesk
---

# Unattended access

Unattended access allows an authorized operator to connect while no user is present to approve the session. Depending on the product and policy, authentication may use a configured password, an assigned account, multifactor authentication, allowlists, or managed-device trust. An attacker who obtains valid reusable authentication material may gain standing access, but configuration files and registry keys do not necessarily contain a recoverable password; storage and protection must be assessed for the exact product and version.

```bash
# inventory TeamViewer configuration; this does not itself recover a password
reg query 'HKLM\SOFTWARE\WOW6432Node\TeamViewer' 2>/dev/null
# review account assignment, unattended-access policy, allowlists, and MFA separately
```

## Exploitation notes

- Valid reusable unattended-access material can provide durable remote entry, but actual access remains subject to the product's account, approval, allowlist, and multifactor policies.
- Local configuration is useful for inventorying IDs and policy. Claim password recovery only when a documented, version-specific storage weakness or credential-extraction method applies.
- Weak or default unattended passwords are guessable directly; reuse means a password found elsewhere may be the unattended one.
- This is the highest-value authentication outcome because it is standing access; combine with [weak passwords](weak-passwords.md) and [ID-based access](id-based-access.md) to reach it.

## References

- [TeamViewer unattended access](https://www.teamviewer.com/en/)
- [AnyDesk unattended access](https://anydesk.com/en/)
