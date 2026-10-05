---
title: "Enumeration: fingerprinting LibreNMS and Observium"
description: "LibreNMS and Observium are identified from their web interface branding and version, exposed in the login page and API. The version decides which authenticated command-injection flaws apply, and distinguishing LibreNMS from Observium matters because their features and vulnerability histories differ despite the shared origin."
keywords:
  - librenms enumeration
  - observium
  - version
  - fingerprint
  - api
---

# Enumeration

Identify which of the two is running and its version. LibreNMS and Observium share a lineage but have diverged, so their features, API, and vulnerabilities differ, and the web interface branding, page structure, and version string distinguish them. LibreNMS exposes a version in the UI and its API; Observium similarly. The version is the fact that decides which authenticated command-injection and other flaws apply to the target, so read it before attacking.

```bash
# distinguish the product and read the version
curl -sk https://<target>/ | grep -ioE 'LibreNMS|Observium'
curl -sk https://<target>/api/v0/system -H 'X-Auth-Token: <token>'   # LibreNMS API (version/system)
curl -sk https://<target>/ | grep -ioE 'version[^<]*[0-9][0-9.]+'
```

## Exploitation notes

- Distinguish LibreNMS from Observium first: their vulnerability histories and features differ despite the common origin, so the product selects the applicable [command-injection](command-injection.md) paths.
- The version maps to the known authenticated RCE flaws; fingerprint precisely.
- LibreNMS has a token-based API that exposes system and device data once authenticated; note it for enumeration after access.
- Route to [authentication](authentication.md), then to command injection and [credential harvesting](credential-harvesting.md).

## References

- [LibreNMS API](https://docs.librenms.org/API/)
- [Observium documentation](https://docs.observium.org/)
