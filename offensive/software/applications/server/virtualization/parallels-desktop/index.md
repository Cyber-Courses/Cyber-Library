---
title: "Parallels Desktop: attacking the macOS desktop hypervisor"
description: "Attacking Parallels Desktop, the mainstream macOS desktop hypervisor: guest-to-host escapes through its device emulation and integration services, and abuse of Shared Folders, Shared Profile, clipboard, and drag-and-drop to reach the macOS host from a guest."
keywords:
  - Parallels Desktop
  - macOS hypervisor
  - VM escape
  - shared folders
  - integration
---

# Parallels Desktop

Parallels Desktop is the dominant hypervisor for running Windows and Linux on macOS. It emulates guest hardware in host-side processes (the escape surface) and offers deep macOS integration (Shared Folders, Shared Profile, clipboard, drag-and-drop) that reaches the host directly. It is a recurring Pwn2Own target, where guest-to-host escapes against it are regularly demonstrated.

## Subtopics

- **[Guest to host escape](guest-to-host-escape.md)**: breaking out through device emulation.
- **[Shared folder and clipboard abuse](shared-folder-and-clipboard-abuse.md)**: reaching host files through integration.
- **[Known escape exploits](known-escape-exploits.md)**: named Parallels breakouts.

## References

- [Parallels Desktop documentation](https://www.parallels.com/products/desktop/resources/)
- [Zero Day Initiative: Parallels research](https://www.zerodayinitiative.com/blog)
