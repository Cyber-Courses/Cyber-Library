---
title: "AFP: attacking Apple Filing Protocol shares"
description: "Attacking the Apple Filing Protocol (AFP) and its common open-source server Netatalk: enumerating and mounting shares with guest access, weak and default credentials, and named remote code execution flaws in Netatalk reachable from an unauthenticated client."
keywords:
  - AFP
  - Apple Filing Protocol
  - Netatalk
  - Time Machine
  - port 548
---

# AFP

The Apple Filing Protocol serves macOS file sharing and Time Machine backups, usually on port 548, most often through the open-source Netatalk server on Linux and NAS devices. Offensive interest is reaching shares without proper credentials (guest access, defaults) and exploiting Netatalk, which has a history of unauthenticated remote code execution.

## Subtopics

- **[Guest access](guest-access.md)**: anonymous mounting and share listing.
- **[Default credentials](default-credentials.md)**: weak and shipped credentials.
- **[Netatalk exploits](netatalk-exploits.md)**: named RCE in the AFP server.

## References

- [Netatalk project](https://netatalk.io/)
- [Apple Filing Protocol overview](https://developer.apple.com/library/archive/documentation/Networking/Conceptual/AFP/Introduction/Introduction.html)
