---
title: "Credential harvesting: stored device and network credentials in Orion"
description: "To poll and manage devices, Orion stores their credentials, SNMP strings, SSH and WMI logins, and API secrets, encrypted in its database with keys on the Orion server. With admin access or server/database compromise, an attacker recovers and decrypts these to obtain working credentials for the whole managed estate, including domain accounts Orion uses for WMI."
keywords:
  - solarwinds credentials
  - orion database
  - snmp wmi
  - domain account
  - harvesting
---

# Credential harvesting

Orion's central role means it stores credentials for everything it manages: SNMP community strings for network devices, SSH logins for network and Unix systems, Windows/WMI credentials (frequently domain accounts) for servers, and API keys for integrations. These are kept in the Orion SQL Server database, encrypted with keys held on the Orion server. An attacker who reaches the database (through [code execution](known-exploits.md), SQL injection, or database access) and the server's key material recovers and decrypts them, yielding working credentials for the entire managed estate. The WMI/Windows credentials are especially valuable because Orion often runs with privileged domain accounts to manage servers, so harvesting Orion's credential store frequently includes high-privilege domain access, in addition to SNMP (reusable to read/reconfigure devices) and SSH logins.

```bash
# with DB access (via RCE/SQLi), the credentials live in the Orion database,
# encrypted with keys on the Orion server; recover both, then decrypt:
#   the Credentials/NodeSettings tables hold encrypted SNMP/SSH/WMI secrets
#   decryption uses the Orion server's key material (DPAPI/config keys)
# admin console access also exposes/edits node credentials per device
```

## Exploitation notes

- Orion's credential store is a master key to the managed estate: SNMP strings (reusable for device read/write), SSH logins, and WMI/Windows credentials (often privileged domain accounts) all come out of it.
- The WMI/domain credentials are the highest value, as Orion commonly manages Windows servers with privileged accounts; harvesting them can yield domain-wide access.
- Decryption requires the Orion server's key material alongside the database, so pair DB access with server access (code execution) for the full recovery.
- This makes Orion worth compromising even without onward RCE; its stored credentials are access to the network and often the domain, so prioritise the harvest.

## References

- [SolarWinds Orion: credential management](https://documentation.solarwinds.com/)
- [SolarWinds security advisories](https://www.solarwinds.com/trust-center/security-advisories)
