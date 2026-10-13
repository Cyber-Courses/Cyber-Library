---
title: "Windows host information: extended data over SNMP"
order: 3
description: "Where the SNMP service is enabled on Windows, the host-resources MIB and the legacy LanMgr MIB expose extended data: local user accounts, running processes, installed software, shares, and listening services. Walking these profiles the host in depth, revealing accounts to target, software to exploit, and the system's role, from a read community string."
keywords:
  - windows snmp
  - host-resources mib
  - lanmgr mib
  - running processes
  - installed software
---

# Windows host information

When the SNMP service is installed and enabled on a Windows host, it exposes far more than the system group through two MIBs. The host-resources MIB (`1.3.6.1.2.1.25`) is standard and rich: `hrSWRunName` lists running processes, `hrSWInstalledName` lists installed software, `hrStorageTable` gives disks and memory, and `hrDeviceTable` the hardware. The legacy LanMgr-MIB-II (`1.3.6.1.4.1.77`) adds Windows-specific data: `.1.2.25` enumerates local user accounts, and other subtrees expose shares and domain/server roles. Walking these profiles the host in depth, local usernames to target for password attacks, running and installed software to match to exploits, shares to access, and the machine's role, all from a read community string without authenticating to the host.

```bash
# host-resources MIB: processes and installed software
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.25.4.2.1.2   # hrSWRunName (running processes)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.25.6.3.1.2   # hrSWInstalledName (installed software)
# LanMgr-MIB: local user accounts and shares
snmpbulkwalk -v2c -c public <target> 1.3.6.1.4.1.77.1.2.25    # local user accounts
snmpbulkwalk -v2c -c public <target> 1.3.6.1.4.1.77.1.2.27    # shares
# snmp-check and snmpenum format the Windows data automatically
snmp-check -c public <target>
```

## Exploitation notes

- The local-user enumeration (`1.3.6.1.4.1.77.1.2.25`) is directly useful: it yields a validated username list for password spraying against SMB, RDP, or WinRM on the same host, obtained without touching those services.
- Running processes and installed software reveal what to attack (a vulnerable service, an EDR to consider, a database) and the host's role; the process list can also expose command lines in some configurations.
- This requires the Windows SNMP feature to be enabled, which is less common on modern systems but frequent on older servers and appliances; `sysDescr` confirming Windows plus a working string is the cue to walk these MIBs.
- Feed the usernames and software into the host's other services; combine with [System and topology disclosure](system-and-topology-disclosure.md) for the network context.

## References

- [RFC 2790 (Host Resources MIB)](https://datatracker.ietf.org/doc/html/rfc2790)
- [HackTricks: SNMP Windows](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
