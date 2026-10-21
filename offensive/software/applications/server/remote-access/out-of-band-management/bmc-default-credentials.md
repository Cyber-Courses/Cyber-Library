---
title: "BMC default credentials: shipped admin logins on management controllers"
order: 3
description: "Some baseboard management controllers and legacy configurations use well-known default credentials, while newer products commonly ship with unique per-device passwords. An unchanged administrative default can grant control of host power, console, and virtual media."
keywords:
  - bmc default credentials
  - idrac root calvin
  - admin admin
  - supermicro
  - management controller
---

# BMC default credentials

Older BMCs and legacy factory configurations often use fixed default credentials, and those credentials may persist because the management controller is treated as isolated infrastructure. Defaults vary by vendor, product generation, and ordering option. Dell's legacy iDRAC configuration uses `root`/`calvin`, but current PowerEdge systems normally ship with a unique generated password unless the legacy option was selected. Other vendors likewise use either model-specific fixed defaults or per-device credentials printed on a label. Successful authentication with an unchanged administrative default can grant control of host power, physical console, and virtual media.

```bash
# identify the vendor and generation, then test its documented default policy
ipmitool -I lanplus -H <bmc> -U root -P calvin user list          # legacy-configured Dell iDRAC
ipmitool -I lanplus -H <bmc> -U ADMIN -P ADMIN user list          # common Supermicro default
curl -sk https://<bmc>/ | grep -iE 'idrac|ilo|supermicro'         # fingerprint for the web UI
```

## Exploitation notes

- Fingerprint the vendor and generation first, then consult that product's documented factory policy. Treat `root`/`calvin` as a Dell legacy option rather than a universal current default.
- The BMC being on a "private" management network is why defaults persist; once you reach that network (a flat segment, a jump host, a VLAN hop), defaults are the easiest path.
- Default admin access yields power control, KVM console, and virtual media, enough to boot an attacker image and own the OS, see [iDRAC and iLO](idrac-and-ilo.md).
- Combine with [RAKP hash disclosure](ipmi-rakp-hash-disclosure.md) (which often recovers exactly these default passwords) and [cipher 0](ipmi-cipher-0-bypass.md) where defaults were changed.

## References

- [Dell iDRAC default username and password guidance](https://www.dell.com/support/kbdoc/en-us/000133536/dell-poweredge-what-is-the-default-username-and-password-for-idrac)
- [HackTricks: IPMI/BMC](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
