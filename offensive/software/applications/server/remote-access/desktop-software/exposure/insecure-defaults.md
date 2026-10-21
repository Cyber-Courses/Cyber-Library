---
title: "Insecure configuration: remote-desktop access controls"
description: "Review whether administrators enabled unattended access, selected weak or reused authentication material, omitted allowlists or multifactor authentication, granted excessive permissions, or weakened interactive approval. Do not describe these choices as vendor defaults without product-specific evidence."
keywords:
  - insecure defaults
  - unattended access
  - allowlist
  - approval prompt
  - configuration
---

# Insecure configuration

Client policy determines which incoming sessions are permitted and what a connected operator may do. Risk increases when unattended access is deliberately enabled with weak or reused authentication material, allowlists and multifactor authentication are omitted, saved access tokens are not revoked, or permission profiles grant more capability than required. These are configuration findings, not universal defaults. AnyDesk, for example, documents that Unattended Access is disabled by default and requires a configured password; interactive requests otherwise require a user decision according to the selected policy.

```bash
# on a host, review the client configuration for the open settings
#   unattended access enabled? allowlist configured? approval required? 2FA on?
#   (locations: TeamViewer/AnyDesk registry and app-data config)
reg query 'HKLM\SOFTWARE\WOW6432Node\TeamViewer' 2>nul      # inspect settings
# record the actual values and compare them with the organization's policy
```

## Exploitation notes

- Unattended access can provide recurring entry when its valid authentication requirements are met; see [unattended access](../authentication/unattended-access.md).
- Confirm whether an allowlist, interactive approval, multifactor authentication, account assignment, or device trust provides an additional gate.
- Review saved access tokens and their revocation behavior as well as passwords; AnyDesk documents device-specific tokens for saved logins.
- Assess the permissions granted after authentication, because successful connection does not necessarily imply unrestricted control.

## References

- [TeamViewer block and allowlist](https://www.teamviewer.com/en/global/support/knowledge-base/teamviewer-remote/security/block-and-allowlist/)
- [AnyDesk client settings](https://support.anydesk.com/docs/settings)
- [AnyDesk unattended access](https://support.anydesk.com/docs/unattended-access)
