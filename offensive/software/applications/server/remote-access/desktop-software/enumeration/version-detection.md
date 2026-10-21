---
title: "Version detection: determining the remote-desktop client version"
description: "Determine the installed remote-desktop product, platform, edition, and complete version from reliable endpoint evidence, then compare them with the exact affected and fixed releases in a named vendor advisory or CVE."
keywords:
  - version detection
  - client version
  - advisory matching
  - teamviewer
  - anydesk
---

# Version detection

Product vulnerabilities and security behavior vary by platform, edition, configuration, and release. Obtain the complete installed version from the application's About view, package inventory, binary metadata, or another documented endpoint source. Then match it to a named vendor advisory or CVE, preserving the advisory's prerequisites and impact. A port number or generic relay connection does not establish a client version, and an undocumented handshake inference should not be treated as an exact fingerprint.

```powershell
# Windows: read version metadata from installed binaries
(Get-Item 'C:\Program Files\TeamViewer\TeamViewer.exe').VersionInfo.ProductVersion
(Get-Item 'C:\Program Files (x86)\AnyDesk\AnyDesk.exe').VersionInfo.ProductVersion
# adjust paths to the installation, and verify the result against the application UI
```

## Exploitation notes

- Record the product, operating system, architecture, edition, and full version because advisories may distinguish among them.
- Map the build to a specific entry in the [known product issues](../known-product-exploits/index.md); do not convert an authorization flaw or credential-coercion issue into a generic bypass or RCE.
- If only network telemetry is available, report the product indication and uncertainty rather than inventing a precise version.
- Combine with [service detection](service-detection.md) to complete the fingerprint before choosing the attack.

## References

- [TeamViewer security bulletins](https://www.teamviewer.com/en/resources/trust-center/security-bulletins/)
- [AnyDesk security](https://anydesk.com/en/security)
