---
title: "Device information: the SNMP system group"
description: "The SNMP system group at 1.3.6.1.2.1.1 returns a device's description, object ID, uptime, contact, name, and location. The sysDescr string typically names the exact OS, model, and firmware, fingerprinting the device for vulnerability matching, and the contact and location fields leak organizational detail useful for targeting and social engineering."
keywords:
  - system group
  - sysdescr
  - sysname
  - syslocation
  - fingerprint
---

# Device information

The system group, OID `1.3.6.1.2.1.1`, is the first thing to read: it returns a compact, high-value identity record present on every SNMP agent. `sysDescr` (`.1.1.0`) is a free-text description that typically names the exact operating system, device model, and firmware or software version, which fingerprints the device precisely for vulnerability matching. `sysObjectID` (`.2.0`) identifies the vendor and product by OID. `sysUpTime` (`.3.0`) gives uptime (useful to infer patch cadence and last reboot). And `sysContact`, `sysName`, and `sysLocation` (`.4`/`.5`/`.6`) are administrator-set fields that frequently leak an admin's name and email, the device's hostname, and a physical location.

```bash
# the whole system group in one walk
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.1
# or the individual high-value OIDs
snmpget -v2c -c public <target> \
  1.3.6.1.2.1.1.1.0 \   # sysDescr  (OS/model/firmware)
  1.3.6.1.2.1.1.4.0 \   # sysContact (admin name/email)
  1.3.6.1.2.1.1.5.0 \   # sysName    (hostname)
  1.3.6.1.2.1.1.6.0     # sysLocation
```

## Exploitation notes

- `sysDescr` is the key fingerprint: it usually states the exact platform and version (for example a Cisco IOS train, a printer firmware, a Windows build), which maps the device to its known vulnerabilities and tells you which deeper OIDs (vendor subtrees) are worth walking.
- `sysContact` and `sysLocation` are reconnaissance and social-engineering fodder: real admin emails and physical locations are common, and the hostname in `sysName` seeds further enumeration.
- This group is readable with any read community string and is the cheapest first step; it also confirms the string works before a full walk.
- Match the identified platform to the right follow-on: Cisco to [Cisco configuration exfiltration](../information-disclosure/cisco-configuration-exfiltration.md), Windows to [Windows host information](../information-disclosure/windows-host-information.md).

## References

- [RFC 1213 (MIB-II system group)](https://datatracker.ietf.org/doc/html/rfc1213)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
