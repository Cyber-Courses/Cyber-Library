---
title: "Guest access: unauthenticated access to AFP volumes"
order: 3
description: "Many AFP servers, especially NAS appliances and Netatalk installs, enable guest access, advertised as the No User Authent capability. A client logging in as guest mounts and browses the exposed volumes with no credentials, reading (and sometimes writing) shared data, making guest-enabled AFP an immediate pre-credential foothold."
keywords:
  - afp guest
  - no user authent
  - netatalk
  - unauthenticated
  - volume access
---

# Guest access

AFP supports a guest login, and it is commonly left enabled: NAS devices ship with it on for convenience, and Netatalk configurations frequently allow it. The server advertises the capability in its `afp-serverinfo` response as the `No User Authent` UAM (User Authentication Module). A client that logs in as guest mounts the exposed volumes and reads their contents, and where the volume permits, writes to them, all without any credential. This makes a guest-enabled AFP server an immediate reconnaissance-and-loot foothold.

```bash
# confirm guest/No-User-Authent is offered
nmap -p548 --script afp-serverinfo <target>    # look for "No User Authent" in UAMs
nmap -p548 --script afp-showmount,afp-ls <target>   # list volumes and contents as guest
# mount a volume as guest (macOS, or afpfs-ng on Linux)
mount_afp afp://;AUTH=No%20User%20Authent@<target>/<Volume> /mnt/afp   # macOS
afp_client mount -u guest <target> <Volume> /mnt/afp                    # afpfs-ng
```

## Exploitation notes

- The `afp-serverinfo` UAM list is the tell: `No User Authent` means guest login is accepted, so test it before anything else.
- Guest access is read at minimum and sometimes write, depending on the volume configuration; browse for the same loot as any file share (configs, keys, backups), see the SMB looting approach for the content patterns.
- NAS appliances and Time Machine targets are the common guest-enabled cases; Time Machine volumes contain full system backups worth pulling.
- Guest access is pre-credential; combine with [default credentials](default-credentials.md) if guest is disabled but the appliance uses weak accounts.

## Tools

- [afpfs-ng (Linux AFP client)](https://sourceforge.net/projects/afpfs-ng/)
- [nmap AFP scripts](https://nmap.org/nsedoc/)

## References

- [Netatalk configuration (afp.conf)](https://netatalk.io/docs)
- [nmap afp-serverinfo](https://nmap.org/nsedoc/scripts/afp-serverinfo.html)
