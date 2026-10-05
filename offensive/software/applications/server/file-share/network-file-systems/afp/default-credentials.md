---
title: "Default credentials: weak AFP authentication"
description: "Reaching AFP shares with weak, default, or guessable credentials shipped by NAS vendors or set weakly by users, and brute-forcing AFP logins, to mount volumes that guest access does not expose."
keywords:
  - AFP default credentials
  - NAS default password
  - AFP brute force
  - weak authentication
  - Netatalk
---

# Default credentials

Where guest access is disabled, AFP still often falls to weak authentication: NAS appliances ship with default administrator credentials, and users set weak passwords on shares. Trying vendor defaults and brute-forcing logins opens the volumes that guest access does not.

```bash
# AFP speaks over 548; brute force with a protocol-aware tool or Metasploit's afp_login
# Try known NAS defaults (admin/admin, admin/password, vendor-specific) first
nmap -p 548 --script afp-brute <target>
```

## Exploitation notes

- NAS defaults are documented per vendor; a quick check of admin/admin and the device's shipped password is often enough.
- Authenticated access exposes all of a user's volumes, including non-guest Time Machine backups.
- Recovered AFP credentials are frequently reused for the device's web admin and other services.

## References

- [Nmap afp-brute](https://nmap.org/nsedoc/scripts/afp-brute.html)
- [Netatalk project](https://netatalk.io/)
