---
title: "Default credentials: vendor and factory SSH logins"
description: "Network appliances, IoT devices, and preinstalled software frequently ship with fixed SSH credentials that are never changed, root:root, admin:admin, vendor-specific pairs, and hardcoded support accounts. Trying the device's documented defaults against SSH is often the fastest authenticated access, and these accounts are commonly administrative."
keywords:
  - default credentials
  - ssh
  - vendor default
  - iot
  - factory password
---

# Default credentials

A large share of SSH-exposed systems are not general-purpose servers but appliances and devices, routers, switches, cameras, NAS units, and embedded Linux products, that ship with fixed SSH credentials. Vendors document these, users rarely change them, and they are frequently administrative or root. So before brute forcing, the highest-yield move against an appliance is trying its known defaults: generic pairs (`root:root`, `admin:admin`, `root:toor`), vendor-specific accounts, and hardcoded support or maintenance logins that some products embed.

```bash
# identify the device/vendor from the banner, then try its documented defaults
nc <target> 22                                 # banner may name the product/firmware
ssh root@<target>        # try root:root, root:<vendor>, admin:admin, etc.
# spray a curated default-credential list across the service
nxc ssh <target> -u defaults_users.txt -p defaults_pass.txt --no-bruteforce   # pair 1:1
hydra -C vendor_defaults.txt ssh://<target>    # combined user:pass list
```

## Exploitation notes

- Fingerprint the device first (banner, associated web UI, firmware strings); default credentials are product-specific, so identification turns a guess into a lookup.
- Use a 1:1 user:password pairing (`-C` in hydra, combo lists) for documented defaults rather than a full cross-product, which is faster and quieter.
- These accounts are often root/administrative on appliances, so success is typically full device control, not a low-privilege shell.
- Hardcoded or undocumented support accounts (a backdoor-by-design) are a known class on some devices; firmware analysis or public advisories reveal them.

## References

- [HackTricks: SSH default credentials](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
- [SecLists default credential lists](https://github.com/danielmiessler/SecLists)
