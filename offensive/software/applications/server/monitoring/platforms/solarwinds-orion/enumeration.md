---
title: "Enumeration: fingerprinting SolarWinds Orion"
order: 4
description: "Orion is identified by its web console (Orion/Login.aspx), page branding, and version in the interface and resources. The version and installed modules decide which known bypass and RCE chains apply, so reading the build before attacking directs the choice between an unauthenticated exploit and the credential route."
keywords:
  - solarwinds enumeration
  - orion login
  - version
  - modules
  - fingerprint
---

# Enumeration

SolarWinds Orion exposes a recognizable web console (the login at `/Orion/Login.aspx`) with SolarWinds branding, and the version appears in the interface, the footer, and static resources. The version and which Orion modules are installed (Network Performance Monitor, Server & Application Monitor, and others) decide which known vulnerabilities apply, since the serious bypass and remote-code-execution chains are version- and module-specific. Reading the build first determines whether an unauthenticated exploit is available for this instance or whether the credential route is the path.

```bash
# fingerprint the console and version
curl -sk https://<target>/Orion/Login.aspx | grep -ioE 'SolarWinds|Orion Platform [0-9.]+'
curl -sk https://<target>/Orion/ | grep -ioE 'version[^<]*[0-9][0-9.]+'
# the Orion API/service endpoints (e.g. /SolarWinds/InformationService/) indicate modules
```

## Exploitation notes

- The Orion Platform version and installed modules map to the applicable [known exploits](known-exploits.md); fingerprint both, as the chains are version/module-specific.
- A reachable console plus an in-range version may be an unauthenticated exploit; otherwise the [authentication](authentication.md) and [credential-harvesting](credential-harvesting.md) routes apply.
- Orion is often internet-exposed in part (web console or API endpoints); identify which components are reachable.
- Route to the exploit chains or to authentication depending on access and version.

## References

- [SolarWinds Orion Platform](https://www.solarwinds.com/)
- [SolarWinds security advisories](https://www.solarwinds.com/trust-center/security-advisories)
