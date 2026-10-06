---
title: "Credential harvesting: stored device credentials in LibreNMS/Observium"
description: "LibreNMS and Observium store the SNMP community strings and SNMPv3 users they use to poll every monitored device, in their databases and configuration. With admin access or code execution, an attacker reads these to obtain working credentials for the whole monitored network, making the platform a single point from which to compromise the devices it watches."
keywords:
  - librenms credentials
  - snmp community
  - device credentials
  - database
  - harvesting
---

# Credential harvesting

To poll devices, both platforms store the credentials to reach them, principally SNMP community strings and SNMPv3 authentication and privacy settings, per device in their databases (the `devices` table in LibreNMS, equivalent in Observium) and in configuration. An attacker with admin access reads them through the UI (each device's SNMP settings) and the API, and with code execution or database access reads them in bulk directly. The harvested strings are immediately reusable against the devices, for SNMP read and often read-write, and any other stored collector credentials pivot further. Because these platforms monitor the whole network, their credential store is effectively a master key to the devices they watch.

```bash
# admin UI/API: per-device SNMP settings are shown and returned
curl -sk -H 'X-Auth-Token: <token>' https://<target>/api/v0/devices | jq '.devices[] | {hostname, community}'
# with DB access (via RCE or SQLi): read the device credentials in bulk
#   LibreNMS:  SELECT hostname, community, snmpver, authname, authpass, authalgo,
#              cryptopass, cryptoalgo FROM devices;
```

## Exploitation notes

- The `devices` table holds each monitored host's SNMP community and v3 credentials; reading it yields working access to the whole monitored network for [SNMP enumeration and write](../../protocols/snmp/index.md).
- Database access (from [command injection](command-injection.md) or SQLi) is the cleanest bulk harvest; the API and UI expose the same per device.
- Harvested read-write community strings enable device reconfiguration (and Cisco config exfiltration); v3 credentials and any other collector logins pivot to those services.
- This makes the platform valuable even without onward RCE: its stored credentials are access to the network it monitors.

## References

- [LibreNMS: device/SNMP configuration](https://docs.librenms.org/)
- [Observium documentation](https://docs.observium.org/)
