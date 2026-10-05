---
title: "AFP: attacking the Apple Filing Protocol"
description: "AFP serves Apple file shares on TCP 548, natively on older macOS and via Netatalk on Linux and NAS devices. The offensive surface is guest access that many servers leave enabled, default and weak credentials on appliances, and the Netatalk implementation's history of serious pre-authentication remote code execution vulnerabilities."
keywords:
  - afp
  - apple filing protocol
  - netatalk
  - port 548
  - guest access
---

# AFP

AFP (Apple Filing Protocol) serves Apple-style file shares on TCP 548. It was macOS's native file sharing before SMB took over, and it persists on older macOS servers, Time Machine targets, and especially on NAS appliances and Linux servers running Netatalk, the open-source AFP implementation. Offensively there are three threads: guest access, which many AFP servers enable by default and which exposes volumes without credentials; default and weak credentials on appliances; and Netatalk's implementation, which has carried several serious pre-authentication remote code execution bugs that make an exposed Netatalk a direct compromise target.

```bash
# discover AFP and enumerate server info/volumes
nmap -p548 --script afp-serverinfo,afp-showmount,afp-ls <target>
# afp-serverinfo reveals the server name, version, and supported auth (incl. "No User Authent" = guest)
```

## Subtopics

- **[Guest access](guest-access.md)**: unauthenticated access to AFP volumes.
- **[Default credentials](default-credentials.md)**: weak and vendor-default accounts on appliances.
- **[Netatalk exploits](netatalk-exploits.md)**: the implementation's pre-auth RCE history.

## References

- [Netatalk project](https://netatalk.io/)
- [nmap AFP scripts](https://nmap.org/nsedoc/)
- [Apple Filing Protocol reference](https://developer.apple.com/library/archive/documentation/Networking/Conceptual/AFP/)
