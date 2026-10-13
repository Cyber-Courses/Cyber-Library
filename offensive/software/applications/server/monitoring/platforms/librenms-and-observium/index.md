---
title: "LibreNMS and Observium: attacking the network monitoring platforms"
order: 5
description: "LibreNMS and Observium are PHP network-monitoring platforms that poll devices over SNMP and store their credentials. The surface is the web authentication, the authenticated command-injection paths that reach code execution on the monitoring host, and harvesting the stored SNMP strings and device credentials that give access to the whole monitored network."
keywords:
  - librenms
  - observium
  - snmp
  - command injection
  - credentials
---

# LibreNMS and Observium

LibreNMS (an actively developed fork of Observium) and Observium are PHP-based network monitoring platforms that discover and poll devices largely over SNMP, storing their credentials to do so. Both are attacked the same way. The web interface authenticates with a password and has default and weak-credential exposure. Once authenticated, both have had command-injection vulnerabilities in features that build shell commands from user input, reaching code execution on the monitoring host. And both store the SNMP community strings and device credentials they use to poll, so harvesting those yields access to every monitored device. The platform compromise therefore cascades into the network it watches.

```bash
curl -sk https://<target>/ | grep -iE 'librenms|observium'       # fingerprint
```

## Subtopics

- **[Enumeration](enumeration.md)**: product and version fingerprinting.
- **[Authentication](authentication.md)**: default and weak credentials.
- **[Command injection](command-injection.md)**: the authenticated RCE paths.
- **[Credential harvesting](credential-harvesting.md)**: stored SNMP strings and device credentials.

## References

- [LibreNMS documentation](https://docs.librenms.org/)
- [LibreNMS security advisories](https://github.com/librenms/librenms/security/advisories)
