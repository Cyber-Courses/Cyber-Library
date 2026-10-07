---
title: "Credential harvesting: stored SNMP strings and device credentials"
order: 3
description: "Cacti stores the credentials it uses to poll devices, SNMP community strings and SNMPv3 users, and credentials for other data collectors, in its database and settings. With admin access or code execution, an attacker reads these to obtain working credentials for every monitored device, turning a Cacti compromise into broad access across the network it watches."
keywords:
  - cacti credentials
  - snmp community
  - data source
  - database
  - harvesting
---

# Credential harvesting

Cacti must authenticate to the devices it polls, so it stores their credentials, and those are a primary prize. SNMP community strings and SNMPv3 user settings are stored per device (and in host templates) to poll them, and data sources for other collection methods hold their own credentials. These live in the Cacti MySQL database (for example the `host` table and settings) and are readable through the admin UI and the API, and directly from the database with code execution or SQL injection. Harvesting them yields working credentials for every monitored device, so a single Cacti compromise becomes access across the estate it watches, the SNMP strings reusable to read (and often write) the devices, and any stored device logins reusable directly.

```bash
# with admin UI access: device SNMP settings are shown per host in the console
#   console > Devices > <host> reveals the SNMP community/version/credentials
# with DB access (via RCE or SQLi), read them directly
#   SELECT description, hostname, snmp_community, snmp_version, snmp_username, snmp_password
#   FROM host;
# settings table holds additional stored secrets
```

## Exploitation notes

- The `host` table (and host templates) hold the per-device SNMP community strings and SNMPv3 credentials Cacti uses to poll; these are immediately reusable against the devices for [SNMP enumeration and write](../../protocols/snmp/index.md).
- Reaching the database (through [code execution](known-exploits.md) or SQLi) gives the cleanest bulk harvest; the admin UI exposes the same per-device.
- The harvested SNMP strings often include read-write strings (for interface/config control), and any other stored data-source credentials pivot further.
- This makes Cacti valuable even without onward RCE: the stored credentials are access to the whole monitored network.

## References

- [Cacti: device and data-source configuration](https://docs.cacti.net/)
- [HackTricks: Cacti](https://book.hacktricks.xyz/)
