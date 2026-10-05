---
title: "Workstation and Fusion: attacking VMware's desktop hypervisors"
description: "Attacking VMware Workstation on Windows and Linux and Fusion on macOS: guest-to-host escapes through the device-emulation code shared with ESXi, and abuse of Shared Folders, drag-and-drop, and clipboard integration to reach the host from a guest."
keywords:
  - VMware Workstation
  - VMware Fusion
  - desktop hypervisor
  - HGFS
  - VM escape
---

# Workstation and Fusion

Workstation and Fusion are VMware's hosted (type-2) hypervisors, running on a user's Windows, Linux, or macOS desktop. They share most of ESXi's `vmware-vmx` device-emulation code, so they share its guest-to-host escape surface, and they add desktop integration features (Shared Folders, drag-and-drop, clipboard) that are their own attack surface. They are common analysis and detonation environments, which makes escaping them valuable.

## Subtopics

- **[Guest to host escape](guest-to-host-escape.md)**: breaking out through shared device emulation.
- **[Shared folder and drag-drop abuse](shared-folder-and-drag-drop-abuse.md)**: reaching host files through integration features.
- **[Known escape exploits](known-escape-exploits.md)**: named desktop VMware breakouts.

## References

- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
