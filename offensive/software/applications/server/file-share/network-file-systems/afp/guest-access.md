---
title: "Guest access: anonymous AFP share access"
description: "Reaching AFP shares without credentials where guest access is enabled: enumerating the server and its volumes and mounting them to read or write files, including Time Machine backups that contain a full copy of a user's system."
keywords:
  - AFP guest
  - anonymous AFP
  - mount_afp
  - Time Machine
  - share enumeration
---

# Guest access

AFP servers frequently allow guest (no-authentication) access to some or all volumes, a convenience default on consumer NAS devices. A guest can enumerate the server's volumes and mount them, reading and sometimes writing files, and Time Machine volumes are especially valuable because they hold a complete, browsable copy of a user's system.

```bash
# Enumerate the server and its volumes as guest
nmap -p 548 --script afp-serverinfo,afp-showmount <target>
# Mount a volume as guest (macOS)
mount_afp 'afp://;AUTH=No%20User%20Authent@<target>/<volume>' /mnt/afp
```

## Exploitation notes

- Time Machine volumes contain the user's files, keychains, and often credentials; mounting one is equivalent to disk access to that Mac.
- `afp-showmount` lists the exported volumes and their access, which flags guest-accessible shares to target.
- Where guest is read-write, a writable volume is a drop point or a poisoning primitive.

## References

- [Nmap AFP scripts](https://nmap.org/nsedoc/scripts/afp-showmount.html)
- [Netatalk project](https://netatalk.io/)
