---
title: "iDRAC and iLO: Dell and HPE controller exploits and virtual media"
order: 4
description: "Dell iDRAC and HPE iLO are vendor BMC stacks with version-specific security advisories. Accounts with the necessary virtual-media, boot, and system-control privileges may be able to mount an image and boot the host from it, subject to platform security controls."
keywords:
  - idrac
  - ilo
  - authentication bypass
  - virtual media
  - remote code execution
---

# iDRAC and iLO

Dell iDRAC and HPE iLO are vendor management-controller stacks built around a BMC. They present two distinct assessment paths. First, both vendors publish version-specific advisories for their management interfaces and APIs, so identify the exact controller model and firmware before determining whether an issue applies. Second, legitimate remote-management features can be abused after obtaining a role with the required privileges. Virtual-media access, boot configuration, and system-control or power privileges may allow an operator to attach an image and attempt to boot the host from it. Success and impact depend on the assigned role, firmware policy, Secure Boot, disk encryption, and the ability to reboot or otherwise change the boot flow.

```bash
# fingerprint the controller and firmware
curl -sS --cacert <ca.pem> https://<bmc>/ | grep -iE 'idrac|integrated lights-out|ilo'
curl -sS --cacert <ca.pem> https://<bmc>/redfish/v1/ | grep -i 'Model\|FirmwareVersion'
# match firmware to the advisory for an auth-bypass/RCE (version-specific)
# with admin access: mount attacker media and boot the host from it
#   iDRAC/iLO virtual media -> attach ISO (SMB/HTTP/local) -> set one-time boot -> power cycle
```

## Exploitation notes

- Two distinct paths require separate evidence: a version-specific product vulnerability may provide some form of controller access, while legitimate console or virtual-media abuse requires an authenticated role with the corresponding privileges. Do not infer administrator access or host compromise from a vulnerable version alone.
- Virtual media plus boot and system-control privileges may let the server start attacker-controlled code. Secure Boot, firmware restrictions, disk encryption, and unavailable reboot control can prevent or materially limit access to the installed OS and its data.
- The console (KVM) additionally gives interactive access to the running OS and its login screen, useful with other credential attacks.
- Controller access can come from [defaults](bmc-default-credentials.md), [recovered RAKP credentials](ipmi-rakp-hash-disclosure.md), [cipher 0](ipmi-cipher-0-bypass.md), or a product vulnerability. Verify the effective role and each required privilege rather than treating every controller session as equivalent to full host control.

## References

- [Dell iDRAC security advisories](https://www.dell.com/support/security/)
- [Dell iDRAC user roles and privileges](https://www.dell.com/support/manuals/en-rs/idrac9-lifecycle-controller-v5.x-series/idrac9_5.00.00.00_ug/idrac-user-roles-and-privileges?guid=guid-d2ed6a79-c1b0-4342-93d1-22c731f6e087&lang=en-us)
- [HPE iLO security advisories](https://support.hpe.com/)
