---
title: "Default community strings: the near-universal SNMP defaults"
description: "SNMP agents ship with default community strings, overwhelmingly public for read and private for read-write, plus vendor-specific defaults. These are rarely changed, especially on printers, switches, and appliances, so trying the documented defaults is the first and highest-yield way to obtain SNMP access."
keywords:
  - default community
  - public
  - private
  - vendor default
  - snmp
---

# Default community strings

SNMP's defaults are the most reliable way in. The overwhelming majority of agents ship with `public` as the read-only string and `private` (or `write`) as the read-write string, and these are left unchanged on a large share of real devices, printers, switches, access points, IP cameras, and appliances especially. Vendors also add their own defaults (for example `cisco`, `ILMI`, `admin`, `security` on various gear). So the first action against any SNMP agent is to try the documented defaults for read and, separately, for write, because a default read string gives full enumeration and a default write string gives device control.

```bash
# test common read defaults
for c in public private cisco ILMI community manager admin security read; do
  snmpget -v2c -c $c -t1 -r0 <target> 1.3.6.1.2.1.1.1.0 2>/dev/null \
    | sed "s/^/[$c] /"; done
# specifically test write (needs a read-write string): a harmless re-set of sysContact
snmpset -v2c -c private <target> 1.3.6.1.2.1.1.4.0 s "$(snmpget -v2c -c private -Ov -Oq <target> 1.3.6.1.2.1.1.4.0 2>/dev/null)" 2>/dev/null && echo "WRITE works: private"
```

## Exploitation notes

- Try read and write defaults separately: `public` for read is near-universal, and a working `private` for write is device control; many devices have one default but not the other.
- Fingerprint the device first where possible ([Device information](../enumeration/device-information.md)) to add the vendor-specific defaults for that product to the list.
- Appliances and embedded devices (printers, cameras, PDUs, switches) are the most likely to retain defaults and are high-value because SNMP often exposes their full configuration.
- If defaults fail, move to [weak strings](weak-community-strings.md) and [brute force](brute-force.md); a found string then drives [enumeration](../enumeration/index.md) and, if read-write, [write access](../write-access.md).

## References

- [SecLists: common SNMP community strings](https://github.com/danielmiessler/SecLists/blob/master/Discovery/SNMP/common-snmp-community-strings.txt)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
