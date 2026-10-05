---
title: "MIB and OID walking: dumping the full SNMP tree"
description: "A full SNMP walk reads every OID the agent exposes, far more than the system group: interfaces, addresses, routing and ARP tables, and vendor subtrees. snmpbulkwalk retrieves it efficiently with GETBULK, and translating numeric OIDs to names with loaded MIBs makes the dump readable, turning a read string into the device's complete published state."
keywords:
  - snmpwalk
  - snmpbulkwalk
  - getbulk
  - mib
  - snmptranslate
---

# MIB and OID walking

Beyond the system group, the agent publishes a large tree, and walking it is how you extract the real content. `snmpwalk` issues successive GETNEXT requests from a starting OID until the subtree ends; `snmpbulkwalk` uses the v2c GETBULK operation to pull many values per request, which is dramatically faster against a device with thousands of OIDs and is the practical default. Numeric OIDs are opaque, so loading MIBs and translating names makes the output readable and reveals which subtrees matter. The standard MIB-II subtrees alone give interfaces (`1.3.6.1.2.1.2`), IP addressing and routing (`1.3.6.1.2.1.4`), and the TCP/UDP connection tables; vendor enterprise subtrees (`1.3.6.1.4.1`) hold the product-specific data.

```bash
# fast full walk (GETBULK); start points for the useful subtrees
snmpbulkwalk -v2c -c public <target>                     # everything
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.2       # interfaces (ifTable)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.21    # ipRouteTable (routing)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.22    # ipNetToMediaTable (ARP)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.4.1         # vendor enterprise subtree
# make it readable: load all MIBs and translate
snmpbulkwalk -v2c -c public -m ALL <target> | head
snmptranslate -On SNMPv2-MIB::sysDescr.0                 # name <-> numeric OID
```

## Exploitation notes

- Use `snmpbulkwalk` (GETBULK) not `snmpwalk` for a full dump; on large devices the difference is minutes versus an hour, and some agents rate-limit the slower GETNEXT loop.
- Target by goal rather than dumping blindly where the device is large: `ifTable` and `ipRouteTable`/`ipNetToMediaTable` give the network picture, the vendor `1.3.6.1.4.1` subtree holds config and credentials, and the host-resources MIB (`1.3.6.1.2.1.25`) gives processes and software on servers.
- Loading MIBs (`-m ALL`, or vendor MIBs) translates numeric OIDs to names so the output is interpretable; `snmptranslate` maps between the two.
- The walk feeds the disclosure pages: topology from the network tables, and the richer loot under [Information disclosure](../information-disclosure/index.md).

## References

- [net-snmp snmpbulkwalk](http://www.net-snmp.org/docs/man/snmpbulkwalk.html)
- [RFC 3416 (GETBULK)](https://datatracker.ietf.org/doc/html/rfc3416)
