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
ipmitool -I lanplus -H <bmc> -U ADMIN -P ADMIN user list          # legacy Supermicro systems only
curl -sS --cacert <ca.pem> https://<bmc>/ | grep -iE 'idrac|ilo|supermicro'
```

## Exploitation notes

- Fingerprint the vendor and generation first, then consult that product's documented factory policy. Treat `root`/`calvin` as a Dell legacy option rather than a universal current default.
- The BMC being on a "private" management network is why defaults persist; once you reach that network (a flat segment, a jump host, a VLAN hop), defaults are the easiest path.
- An unchanged administrative default may provide power, console, boot, and virtual-media privileges. Whether those capabilities lead to OS compromise depends on the account role and controls such as Secure Boot, disk encryption, and boot policy; see [iDRAC and iLO](idrac-and-ilo.md).
- [RAKP hash disclosure](ipmi-rakp-hash-disclosure.md) can enable offline guessing of any weak password. [Cipher 0](ipmi-cipher-0-bypass.md) is a separate configuration weakness: its presence and impact depend on cipher-suite acceptance and the valid account's channel privilege, not on whether its password is a factory default.

## References

- [Dell iDRAC default username and password guidance](https://www.dell.com/support/kbdoc/en-us/000133536/dell-poweredge-what-is-the-default-username-and-password-for-idrac)
- [Supermicro BMC unique password policy](https://www.supermicro.com/en/support/BMC_Unique_Password)
- [HackTricks: IPMI/BMC](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
