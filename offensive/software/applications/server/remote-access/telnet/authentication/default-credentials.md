---
title: "Default credentials: vendor and factory Telnet logins"
description: "Telnet remains present on some network equipment, embedded systems, and legacy hosts. After identifying the exact product and firmware, an assessor can check whether its documented factory credentials still work and then verify the account's actual privileges."
keywords:
  - default credentials
  - telnet
  - iot
  - vendor default
  - factory password
---

# Default credentials

Telnet remains present on some network equipment, embedded systems, and legacy hosts. Factory credential policy varies by vendor, model, firmware, provisioning option, and product age: some systems used fixed defaults, others require first-boot setup or ship with unique credentials. Identify the product before testing a documented factory credential. A successful login establishes only that the credential works; determine the account's actual role and whether an additional enable or privilege step is required before describing its impact.

```bash
# identify the device, then test only documented defaults that apply to it
nc <target> 23
hydra -C product-default-combos.txt telnet://<target> # authorized user:pass list
```

## Exploitation notes

- Fingerprint the device first ([banner](../enumeration/banner-grabbing.md)); a default applies only when its vendor, model, firmware, and provisioning assumptions match the target.
- Prefer a short, evidence-based set of documented pairs to generic credential lists. This reduces lockouts, noise, and false assumptions about current products.
- Verify authorization after login. A Telnet account may open a restricted menu or unprivileged CLI, require an enable step, or map to an administrative or root context.
- Test a blank password only when product documentation or observed configuration makes it a supported candidate.

## References

- [HackTricks: Telnet default credentials](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [SecLists default credentials](https://github.com/danielmiessler/SecLists)
