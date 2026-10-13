---
title: "iDRAC and iLO: Dell and HPE controller exploits and virtual media"
order: 4
description: "Dell iDRAC and HPE iLO are the vendor BMC stacks, and both have had serious flaws: authentication bypasses and remote code execution reachable on the management interface. Beyond exploits, their legitimate virtual-media and console features let an attacker with access mount an attacker ISO and boot the host from it, compromising the OS directly."
keywords:
  - idrac
  - ilo
  - authentication bypass
  - virtual media
  - remote code execution
---

# iDRAC and iLO

Dell iDRAC and HPE iLO are the vendor management controllers built on the BMC, and they are attacked two ways. First, product vulnerabilities: both have had authentication-bypass and remote-code-execution flaws reachable on the management web interface and APIs (iLO notably had an authentication bypass allowing unauthenticated admin access, and iDRAC has had unauthenticated RCE), so fingerprinting the controller and firmware version maps it to its exploit. Second, and independent of any bug, their legitimate features are themselves the weapon: with administrative access (from defaults, cracked IPMI hashes, or an exploit), the virtual-media feature mounts an attacker-supplied ISO and the controller boots the host from it, giving direct compromise of the server's OS.

```bash
# fingerprint the controller and firmware
curl -sk https://<bmc>/ | grep -iE 'idrac|integrated lights-out|ilo'
curl -sk https://<bmc>/redfish/v1/ | grep -i 'Model\|FirmwareVersion'
# match firmware to the advisory for an auth-bypass/RCE (version-specific)
# with admin access: mount attacker media and boot the host from it
#   iDRAC/iLO virtual media -> attach ISO (SMB/HTTP/local) -> set one-time boot -> power cycle
```

## Exploitation notes

- Two distinct paths: a product exploit (auth bypass or RCE, version-specific, fingerprint the firmware and match the advisory) gives controller access without credentials; and legitimate virtual-media/console abuse, once you have admin access by any means, compromises the host OS.
- Virtual media is the decisive feature: mounting an attacker ISO and setting a one-time boot lets the server boot into attacker-controlled code (a live OS to read/modify the disk, reset local admin, or implant), bypassing all OS-level defenses.
- The console (KVM) additionally gives interactive access to the running OS and its login screen, useful with other credential attacks.
- Controller admin access comes from [defaults](bmc-default-credentials.md), [cracked RAKP hashes](ipmi-rakp-hash-disclosure.md), [cipher 0](ipmi-cipher-0-bypass.md), or an exploit; any of them plus virtual media is full host compromise.

## References

- [Dell iDRAC security advisories](https://www.dell.com/support/security/)
- [HPE iLO security advisories](https://support.hpe.com/)
