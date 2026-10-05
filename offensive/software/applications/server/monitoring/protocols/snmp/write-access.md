---
title: "Write access: reconfiguring a device with a read-write community string"
description: "A read-write SNMP community string lets an attacker issue snmpset to change device settings: shutting interfaces, altering routes and ARP entries, modifying the system fields, and, on supported devices, triggering a configuration copy to or from TFTP. This ranges from disruption to planting a backdoored configuration, turning a write string into device control."
keywords:
  - snmpset
  - read-write community
  - config change
  - ifadminstatus
  - config overwrite
---

# Write access

A read-write community string moves SNMP from reading to controlling the device. `snmpset` writes OIDs, and depending on the device's writable MIBs this ranges from nuisance to takeover. Standard writable objects let an attacker change the system fields (`sysContact`/`sysLocation`/`sysName`), administratively shut or enable interfaces (`ifAdminStatus`), and modify routing and ARP entries, causing targeted disruption or traffic redirection. On network devices with the config-copy MIB, write access additionally copies a configuration to a TFTP server (exfiltration) or, in reverse, copies an attacker-supplied configuration from TFTP onto the device, overwriting the running or startup config with a backdoored one. That last capability makes a read-write string equivalent to administrative control.

```bash
RW=private
# change a system field (proof of write, and config tampering)
snmpset -v2c -c $RW <target> 1.3.6.1.2.1.1.6.0 s "owned"        # sysLocation
# administratively shut an interface (disruption / traffic steering)
snmpset -v2c -c $RW <target> 1.3.6.1.2.1.2.2.1.7.<ifIndex> i 2  # ifAdminStatus = down(2)
# Cisco: copy an ATTACKER config FROM tftp ONTO the device (backdoor the config)
#   same CISCO-CONFIG-COPY-MIB as exfiltration, with SourceFileType=networkFile(1)
#   and DestFileType=runningConfig(4) -> loads attacker's cfg into running-config
```

## Exploitation notes

- Confirm the write string with a harmless `snmpset` (re-setting `sysLocation` to its current value) before anything disruptive; read and write strings differ, so a read string does not imply write.
- The highest-impact use on Cisco and similar gear is the config-copy MIB: exfiltrate the config for credentials ([Cisco configuration exfiltration](information-disclosure/cisco-configuration-exfiltration.md)), or load an attacker config to add an account, open management access, or redirect traffic, which is full device compromise.
- Interface and routing writes enable targeted denial of service and man-in-the-middle (shutting a link, injecting a route), useful but noisier and more disruptive than credential theft.
- Writable objects vary by device and MIB; walk for writable OIDs and consult the device's MIBs, and remember config changes may be logged and may require a `write memory` equivalent to persist.

## References

- [net-snmp snmpset](http://www.net-snmp.org/docs/man/snmpset.html)
- [HackTricks: SNMP write](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
