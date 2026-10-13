---
title: "Cacti: attacking the graphing and monitoring platform"
order: 4
description: "Cacti is a PHP network graphing and monitoring application. It has a recurring history of serious vulnerabilities, unauthenticated and authenticated command injection and SQL injection reaching remote code execution, and it stores the SNMP strings and device credentials it uses to poll. Default credentials, the known RCE chains, and credential harvesting are the surface."
keywords:
  - cacti
  - php
  - command injection
  - sql injection
  - snmp
---

# Cacti

Cacti is an open-source PHP application for polling devices (largely over SNMP) and graphing the results, commonly deployed under `/cacti/`. It matters offensively for two reasons: it has a persistent history of serious web vulnerabilities, unauthenticated and authenticated command injection and SQL injection in its data-collection and graphing components, several reaching remote code execution on the server, and it stores the credentials it uses to poll devices (SNMP community strings, and SSH/other credentials for data sources), so compromising it harvests estate credentials. The surface is the login (default `admin`/`admin`), the known exploit chains, and the stored credential harvest.

```bash
curl -sk https://<target>/cacti/ | grep -ioE 'Version [0-9.]+'   # fingerprint
```

## Subtopics

- **[Enumeration](enumeration.md)**: product and version fingerprinting.
- **[Authentication](authentication.md)**: the default admin and weak credentials.
- **[Known exploits](known-exploits.md)**: the SQLi and command-injection RCE chains.
- **[Credential harvesting](credential-harvesting.md)**: stored SNMP strings and device credentials.

## References

- [Cacti documentation](https://docs.cacti.net/)
- [Cacti security advisories](https://github.com/Cacti/cacti/security/advisories)
