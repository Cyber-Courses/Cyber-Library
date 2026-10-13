---
title: "BMC default credentials: shipped admin logins on management controllers"
order: 3
description: "Baseboard management controllers ship with well-known default credentials, ADMIN/ADMIN on many Supermicro boards, root/calvin on Dell iDRAC, and vendor equivalents, that are rarely changed because the BMC is on a separate management network. These defaults grant full administrative control of the host's power, console, and virtual media."
keywords:
  - bmc default credentials
  - idrac root calvin
  - admin admin
  - supermicro
  - management controller
---

# BMC default credentials

BMCs ship with fixed default credentials, and they are changed far less often than OS or application passwords because the management controller lives on a separate network administrators consider private and forget about. The defaults are well known per vendor: `root`/`calvin` on Dell iDRAC, `ADMIN`/`ADMIN` on many Supermicro boards, `Administrator` with a per-device password on HPE iLO (sometimes printed on a pull-tab and left unchanged), and others. Trying these grants full administrative control of the controller, and therefore of the host's power, physical console, and virtual media, so a reachable BMC with default credentials is a complete server compromise.

```bash
# identify the BMC vendor, then try its documented defaults over IPMI and the web UI
ipmitool -I lanplus -H <bmc> -U root -P calvin user list          # Dell iDRAC default
ipmitool -I lanplus -H <bmc> -U ADMIN -P ADMIN user list          # common Supermicro default
curl -sk https://<bmc>/ | grep -iE 'idrac|ilo|supermicro'         # fingerprint for the web UI
```

## Exploitation notes

- Fingerprint the vendor first (web UI, IPMI device ID), then try that vendor's documented defaults; `root`/`calvin` (Dell), `ADMIN`/`ADMIN` (Supermicro) are the highest-yield.
- The BMC being on a "private" management network is why defaults persist; once you reach that network (a flat segment, a jump host, a VLAN hop), defaults are the easiest path.
- Default admin access yields power control, KVM console, and virtual media, enough to boot an attacker image and own the OS, see [iDRAC and iLO](idrac-and-ilo.md).
- Combine with [RAKP hash disclosure](ipmi-rakp-hash-disclosure.md) (which often recovers exactly these default passwords) and [cipher 0](ipmi-cipher-0-bypass.md) where defaults were changed.

## References

- [Dell iDRAC default credentials](https://www.dell.com/support/)
- [HackTricks: IPMI/BMC](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
