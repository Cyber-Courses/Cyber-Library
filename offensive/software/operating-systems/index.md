---
title: "Operating Systems: offensive techniques against the host platform"
description: "Offensive techniques against the operating system itself, the platform that hosts every application: local privilege escalation, credential access, persistence, and kernel exploitation, organized by platform because Windows, Linux, and macOS expose fundamentally different privilege models, credential stores, and system services."
keywords:
  - operating system attacks
  - local privilege escalation
  - credential access
  - persistence
  - kernel exploitation
---

# Operating Systems

Where [Applications](../applications/index.md) covers the code an organization writes, this area covers the platform underneath it: the kernel, system services, drivers, and the privilege and isolation model that everything above depends on. This is the work that begins once you already have code running on a host, usually as an unprivileged user, and want to become `root` or `SYSTEM`, read the credentials the machine holds, survive a reboot, and defeat the boundaries the OS enforces.

The area is organized **by platform**, because the attack surface is platform-specific: the privilege model, the credential stores, the service and driver architecture, and the local isolation features differ fundamentally between the three major operating systems, and so do the techniques and tools that attack them.

## How the host is attacked

The same goals recur on every platform, reached through platform-specific mechanisms:

- **Local privilege escalation**: moving from an unprivileged user to full control of the machine, through misconfiguration, abusable privileges, vulnerable services and drivers, or a kernel flaw.
- **Credential access**: pulling passwords, hashes, keys, and tokens out of memory, the registry or keychain, and on-disk stores to reuse elsewhere.
- **Persistence**: surviving logout and reboot through the platform's own startup, scheduling, and service mechanisms.
- **Isolation and boundaries**: defeating the platform's local controls (UAC, TCC, SIP, Gatekeeper, sandboxes, namespaces).

## Platforms

- **[Windows](windows/index.md)**: the dominant enterprise endpoint and server platform, with the richest local attack surface.
- **[Linux](linux/index.md)**: the dominant server and infrastructure platform, attacked through its Unix privilege model and service configuration.
- **[macOS](macos/index.md)**: the Apple desktop platform, with its own layered privacy, signing, and integrity controls.

## Scope and seams

This area is the **local** host attack surface. Active Directory, which is how Windows hosts trust and authenticate each other across a domain, is attacked under [Directory](../applications/server/directory/index.md); and network-reachable services on a host are under the [Server](../applications/server/index.md) service areas. Those are cross-referenced from here rather than duplicated.

## References

- [MITRE ATT&CK: Privilege Escalation](https://attack.mitre.org/tactics/TA0004/)
- [MITRE ATT&CK: Credential Access](https://attack.mitre.org/tactics/TA0006/)
- [MITRE ATT&CK: Persistence](https://attack.mitre.org/tactics/TA0003/)
