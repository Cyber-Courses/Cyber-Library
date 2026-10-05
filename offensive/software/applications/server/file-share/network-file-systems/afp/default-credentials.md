---
title: "Default credentials: weak and vendor accounts on AFP appliances"
description: "AFP is most often found on NAS appliances that ship with default administrative accounts and weak password policies. Trying vendor default credentials and spraying common passwords against the AFP authentication, or the appliance's web admin, yields authenticated share access and frequently device administration, since the same accounts span both."
keywords:
  - default credentials
  - nas appliance
  - afp
  - password spray
  - netatalk
---

# Default credentials

AFP's typical home is a NAS appliance (and some Linux/Netatalk servers), and appliances are shipped with default accounts and lax password policies. When guest access is disabled, the next move is credentials: vendor defaults and common passwords, tried against the AFP authentication directly and against the appliance's web administration, which usually shares the same user store. A hit gives authenticated access to the shares and often administration of the device, since the admin account spans file access and management.

```bash
# enumerate the auth modules offered (which UAMs/credentials are accepted)
nmap -p548 --script afp-serverinfo <target>
# try credentials against AFP (afp-brute, or a client login)
nmap -p548 --script afp-brute --script-args userdb=users.txt,passdb=pass.txt <target>
# the appliance web admin frequently shares the account store
#   try vendor defaults: admin/admin, admin/<blank>, admin/password, root/<vendor>
curl -sk https://<target>/   # identify the NAS vendor/model to look up its defaults
```

## Exploitation notes

- Identify the appliance vendor and model first (web UI banner, `afp-serverinfo` server name); default credentials are vendor-specific and well documented.
- The AFP user store and the web-admin user store are usually the same on a NAS, so a credential that works for one typically works for the other, turning file access into device administration (and vice versa).
- Keep spraying conservative against any lockout policy; NAS devices vary, and some have no lockout at all, which favours a broader password list.
- Device administration on a NAS often allows enabling SSH, adding users, or reading all shares, escalating a file-share foothold to full appliance control.

## References

- [Netatalk authentication (UAMs)](https://netatalk.io/docs)
- [nmap afp-brute](https://nmap.org/nsedoc/scripts/afp-brute.html)
