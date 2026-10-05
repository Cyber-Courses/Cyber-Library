---
title: "System and topology disclosure: mapping the network over SNMP"
description: "The standard MIB-II tables reconstruct the internal network from a single device: interfaces and their addresses, the routing table, and the ARP (IP-to-MAC) table. Walking these on a router or switch maps adjacent subnets, gateways, and live hosts, giving an attacker the internal topology without scanning, from read-only SNMP access."
keywords:
  - topology
  - iftable
  - iproutetable
  - arp table
  - network map
---

# System and topology disclosure

A single reachable network device, a router, switch, or firewall, discloses much of the surrounding network through the standard MIB-II tables, so a read community string becomes a map of the environment with no active scanning. The interface table (`ifTable`, `1.3.6.1.2.1.2`) and the IP address table (`ipAddrTable`, `1.3.6.1.2.1.4.20`) give the device's interfaces and the subnets it sits on. The routing table (`ipRouteTable`/`ipCidrRouteTable`) reveals gateways and reachable networks, outlining the topology beyond the device. And the ARP/neighbor table (`ipNetToMediaTable`, `1.3.6.1.2.1.4.22`) lists IP-to-MAC mappings, which is a ready-made inventory of live hosts on the device's segments.

```bash
# interfaces and the addresses/subnets the device is on
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.2.2.1.2     # ifDescr (interface names)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.20.1.1    # ipAdEntAddr (IP addresses)
# routing table: gateways and reachable networks
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.21.1      # ipRouteTable
# ARP table: live IP->MAC neighbours (a host inventory)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.22.1.3    # ipNetToMediaNetAddress
# snmp-check formats much of this automatically
snmp-check -c public <target>
```

## Exploitation notes

- A router or layer-3 switch is the best target: its routing table outlines the whole internal topology and its ARP table enumerates live hosts on each connected segment, which is reconnaissance you would otherwise get only by scanning.
- The ARP/neighbor table is effectively a passive host-discovery list for the device's subnets; cross-reference it with the routing table to prioritise where to pivot.
- This is read-only and quiet (no scanning of the hosts themselves), so it maps the network before you touch it.
- Combine with the richer per-host loot: [Windows host information](windows-host-information.md) on servers and [Cisco configuration exfiltration](cisco-configuration-exfiltration.md) on Cisco gear.

## References

- [RFC 1213 (MIB-II IP group)](https://datatracker.ietf.org/doc/html/rfc1213)
- [HackTricks: SNMP topology](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
