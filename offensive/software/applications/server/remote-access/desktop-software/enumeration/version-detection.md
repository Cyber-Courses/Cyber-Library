---
title: "Version detection: determining the remote-desktop client version"
description: "The client version decides which attacks apply: brute-force feasibility (vendors added rate-limiting over time), stored-credential recoverability (storage formats changed), and the specific product vulnerabilities a build is exposed to. Version is read from the client UI, installed files, registry, or handshake, and matched to known advisories."
keywords:
  - version detection
  - client version
  - advisory matching
  - teamviewer
  - anydesk
---

# Version detection

Version is decisive for these products because so much changed across releases. Vendors added connection rate-limiting and lockout after mass account-takeover campaigns, so whether online [password brute force](../authentication/weak-passwords.md) is feasible depends on the version. The format and recoverability of stored credentials (for [unattended access](../authentication/unattended-access.md)) changed across versions. And each product's specific authentication-bypass and code-execution vulnerabilities apply only to particular builds. Version is read from the client UI/About, the installed binary's metadata, registry/app-data, or in some cases the connection handshake, then matched to advisories.

```bash
# host: read the client version from binary metadata / registry
(Get-Item 'C:\Program Files\TeamViewer\TeamViewer.exe').VersionInfo.ProductVersion   # PowerShell
reg query 'HKLM\SOFTWARE\WOW6432Node\TeamViewer' /v Version 2>nul
# AnyDesk version from its binary or app-data
# match the build to the vendor advisory for the applicable exploit
```

## Exploitation notes

- The version gates practicality: an old build may allow online brute force that a current one rate-limits, and may store unattended credentials in a recoverable form that newer versions protect.
- Map the build to product advisories for the [known exploits](../known-product-exploits/index.md) (auth bypass, RCE); these are version-specific.
- Where you only have network access, handshake/relay behaviour may hint at the version; on a host, binary metadata and registry give it exactly.
- Combine with [service detection](service-detection.md) to complete the fingerprint before choosing the attack.

## References

- [TeamViewer release/security notes](https://www.teamviewer.com/en/trust-center/security/)
- [AnyDesk security advisories](https://anydesk.com/en/security)
