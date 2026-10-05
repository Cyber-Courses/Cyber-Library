---
title: "SolarWinds Orion: attacking the network management platform"
description: "SolarWinds Orion is an enterprise network-management platform that polls and manages devices across the estate and stores their credentials. The surface is the web console authentication, the harvested device and network credentials it holds, and the known pre-authentication bypass and remote-code-execution chains in the Orion platform, any of which turns it into broad control of the managed environment."
keywords:
  - solarwinds
  - orion
  - npm
  - credentials
  - rce
---

# SolarWinds Orion

SolarWinds Orion is a widely deployed enterprise platform for network and systems management (Network Performance Monitor and the surrounding modules). It is high-value because it both reaches into and stores credentials for the whole managed estate: it polls and configures devices over SNMP, SSH, WMI, and APIs, holding those credentials, and it sits centrally with broad network access. The attack surface is the web console authentication, harvesting the stored device and network credentials (which unlock the managed environment directly), and the known serious vulnerabilities in the Orion platform, including pre-authentication bypass and remote-code-execution chains. Any of these converts an Orion compromise into control over the devices and systems it manages. Orion is also the platform behind a major supply-chain compromise, underscoring its central position and value.

```bash
curl -sk https://<target>/Orion/Login.aspx | grep -ioE 'SolarWinds[^<]*'   # fingerprint
```

## Subtopics

- **[Enumeration](enumeration.md)**: product and version fingerprinting.
- **[Authentication](authentication.md)**: default and weak console credentials.
- **[Credential harvesting](credential-harvesting.md)**: stored device and network credentials.
- **[Known exploits](known-exploits.md)**: the pre-auth bypass and RCE chains.

## References

- [SolarWinds documentation](https://documentation.solarwinds.com/)
- [SolarWinds security advisories](https://www.solarwinds.com/trust-center/security-advisories)
