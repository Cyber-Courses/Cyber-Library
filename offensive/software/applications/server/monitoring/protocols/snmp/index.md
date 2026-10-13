---
title: "SNMP: attacking the Simple Network Management Protocol"
order: 1
description: "SNMP on UDP 161 exposes a device's entire state through a tree of OIDs, gated only by a community string on v1 and v2c. The attack surface is obtaining that string (default, weak, brute-forced), walking the agent to disclose system, network, and credential data, exfiltrating device configuration, and, with a read-write string, reconfiguring the device."
keywords:
  - snmp
  - udp 161
  - community string
  - oid
  - mib
---

# SNMP

SNMP (Simple Network Management Protocol) runs on UDP 161 and exposes a managed device's state as a tree of object identifiers (OIDs) organized in MIBs. On the still-dominant SNMPv1 and v2c, the only access control is a community string sent in cleartext, a read string (conventionally `public`) for querying and a read-write string (conventionally `private`) for changing settings. That weak model makes SNMP one of the most productive reconnaissance and credential-exposure surfaces on a network: routers, switches, firewalls, printers, servers, and appliances all answer it, and what they return includes the system inventory, the network topology, and frequently credentials and whole device configurations. With a read-write string, SNMP also reconfigures the device.

```bash
# detect and version the agent
nmap -sU -p161 -sV --script snmp-info <target>
snmpwalk -v2c -c public <target> 2>/dev/null | head     # does a read string work?
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: walking the agent and reading device data.
- **[Community strings](community-strings/index.md)**: obtaining read and read-write strings.
- **[Information disclosure](information-disclosure/index.md)**: system, topology, config, and credential exposure.
- **[Versions](versions/index.md)**: v1/v2c cleartext weaknesses and attacking v3.
- **[Write access](write-access.md)**: reconfiguring the device with a read-write string.

## References

- [RFC 3416 (SNMP protocol operations)](https://datatracker.ietf.org/doc/html/rfc3416)
- [HackTricks: SNMP (161)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
- [net-snmp tools](http://www.net-snmp.org/)
