---
title: "Enumeration: walking an SNMP agent"
description: "SNMP enumeration reads the agent's OID tree with a valid community string: the basic system group for device identity, then a full walk of the MIB to dump everything the agent exposes. The walk is where the value is, so efficient bulk walking and targeting the right vendor subtrees turn a read string into a complete picture of the device."
keywords:
  - snmp enumeration
  - snmpwalk
  - snmpbulkwalk
  - mib
  - oid
---

# Enumeration

Enumeration is the core SNMP action: with a working read community string, walk the agent's OID tree and read back what it exposes. Two levels matter. The basic system group (`1.3.6.1.2.1.1`) gives device identity, a description, name, location, contact, and uptime, enough to fingerprint the device. The full walk then dumps everything the agent publishes, which on a real device is extensive: interfaces, addresses, routes, ARP tables, processes, software, and vendor-specific data. The walk is where the payoff is, so doing it efficiently (bulk operations) and knowing which subtrees to target is the skill.

```bash
# basic identity, then the full tree
snmpget -v2c -c public <target> 1.3.6.1.2.1.1.1.0        # sysDescr (one OID)
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.1           # system group
snmpbulkwalk -v2c -c public <target>                     # full walk, far faster than snmpwalk
snmp-check -c public <target>                            # formatted summary of common data
```

## Subtopics

- **[Device information](device-information.md)**: the system group and device identity.
- **[MIB and OID walking](mib-and-oid-walking.md)**: dumping the full tree efficiently.

## References

- [net-snmp snmpwalk/snmpbulkwalk](http://www.net-snmp.org/docs/man/snmpbulkwalk.html)
- [HackTricks: SNMP enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
