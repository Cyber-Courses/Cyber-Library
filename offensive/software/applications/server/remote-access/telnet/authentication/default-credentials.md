---
title: "Default credentials: vendor and factory Telnet logins"
description: "Telnet survives mainly on devices, routers, switches, cameras, printers, and IoT, that ship with fixed default credentials users rarely change. Trying the device's documented defaults, and generic pairs like root:root and admin:admin, is the highest-yield Telnet attack, and the accounts are typically administrative on the device."
keywords:
  - default credentials
  - telnet
  - iot
  - vendor default
  - factory password
---

# Default credentials

Telnet is now mostly a device protocol, and devices ship with fixed default credentials that are documented, widely known, and seldom changed. So the first and highest-yield Telnet attack is trying defaults: generic pairs (`root:root`, `admin:admin`, `root:<vendor>`, blank passwords) and the specific documented credentials for the identified device. These accounts are usually administrative, an enable-capable network account or a root shell on embedded Linux, so a default-credential hit is typically full device control, and mass IoT compromises have been built on exactly this.

```bash
# identify the device (banner), then try its documented defaults + generics
nc <target> 23
hydra -C iot-default-combos.txt telnet://<target>     # user:pass combo list
nxc telnet <target> -u defaults_u.txt -p defaults_p.txt --no-bruteforce   # paired
```

## Exploitation notes

- Fingerprint the device first ([banner](../enumeration/banner-grabbing.md)); defaults are vendor/model-specific, so identification turns guessing into a lookup, and generic pairs cover the rest.
- Use paired combo lists (documented user:pass) rather than full cross-products for speed and quiet; IoT default lists (as used by Telnet-spreading worms) are highly effective.
- The accounts are commonly administrative, so success is device control (config change, firmware, often a root shell), not a low-privilege foothold.
- Blank or empty passwords are a frequent device default; test them explicitly.

## References

- [HackTricks: Telnet default credentials](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [SecLists default credentials](https://github.com/danielmiessler/SecLists)
