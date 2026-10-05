---
title: "Information disclosure: what an SNMP walk reveals"
description: "A read community string turns into real value through what the agent discloses: the system and network topology, a device's full running configuration (exfiltrated on Cisco via SNMP and TFTP), extended Windows host data through the host-resources and LanMgr MIBs, and credentials and secrets stored in standard and custom OIDs."
keywords:
  - snmp disclosure
  - topology
  - running config
  - credentials
  - host resources
---

# Information disclosure

Enumeration is the mechanism; disclosure is the payoff. What an SNMP agent returns to a read string is frequently sensitive and sometimes catastrophic. The network tables reconstruct the internal topology, who is connected, routing, and ARP mappings, mapping the environment from a single device. On network gear, the running configuration itself can be pulled over SNMP, exposing every credential and key in it. On Windows hosts, extended MIBs enumerate users, processes, installed software, and shares. And credentials and secrets, other community strings, wireless and VPN keys, and application data, turn up in standard and vendor OIDs. This section covers turning read access into that loot.

## Subtopics

- **[System and topology disclosure](system-and-topology-disclosure.md)**: mapping the network from the device tables.
- **[Cisco configuration exfiltration](cisco-configuration-exfiltration.md)**: pulling the running config over SNMP and TFTP.
- **[Windows host information](windows-host-information.md)**: users, processes, software, and shares.
- **[Credential and secret exposure](credential-and-secret-exposure.md)**: keys and credentials in OIDs.

## References

- [RFC 1213 (MIB-II)](https://datatracker.ietf.org/doc/html/rfc1213)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
