---
title: "VirtualBox: attacking the Oracle desktop hypervisor"
description: "Attacking Oracle VirtualBox: guest-to-host escapes through its emulated devices (network, graphics, storage), abuse of Shared Folders and clipboard integration to reach host files, and the named breakout exploits repeatedly demonstrated against it."
keywords:
  - VirtualBox
  - VM escape
  - shared folders
  - device emulation
  - Oracle VM
---

# VirtualBox

VirtualBox is Oracle's hosted (type-2) hypervisor, widely used on desktops and in analysis labs. It emulates a broad set of devices in a host process, which is its guest-to-host escape surface, and it offers Shared Folders, clipboard, and drag-and-drop integration that reach the host directly. Its large device surface has made it a frequent target of public escape research.

## Subtopics

- **[Guest to host escape](guest-to-host-escape/index.md)**: breaking out through emulated devices.
- **[Shared folder and clipboard abuse](shared-folder-and-clipboard-abuse.md)**: reaching host files through integration.
- **[Known escape exploits](known-escape-exploits.md)**: named VirtualBox breakouts.

## References

- [Oracle VirtualBox manual](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox research](https://www.zerodayinitiative.com/blog)
