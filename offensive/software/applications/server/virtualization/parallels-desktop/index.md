---
title: "Parallels Desktop: attacking the macOS desktop hypervisor"
description: "Parallels Desktop runs VMs on macOS, with device emulation and integration services in host-side processes. Guest-to-host escapes target its emulated devices and the Parallels Tools integration, shared folders, clipboard, and drag-and-drop over the Parallels guest-host communication channels. It is a recurring Pwn2Own target on the macOS host."
keywords:
  - parallels desktop
  - macos
  - parallels tools
  - shared folders
  - prl
---

# Parallels Desktop

Parallels Desktop is the leading macOS desktop hypervisor, running Windows and Linux VMs with device emulation and integration services hosted in Parallels' own processes (the `prl_*` family) on the macOS host. A guest-to-host escape corrupts one of those host-side components and runs on macOS. The attack surface is the emulated device set and, prominently, the Parallels Tools integration, shared folders, the shared clipboard, and drag-and-drop, carried over Parallels' guest-host communication channels. Parallels is a recurring Pwn2Own target, with escapes repeatedly found in both the device models and the Tools integration.

```bash
# from the guest: emulated devices and Parallels Tools
lspci -nn 2>/dev/null; lsusb 2>/dev/null         # Linux guest
# Parallels Tools present (guest integration drivers/services)
lsmod 2>/dev/null | grep -i prl
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape.md)**: breaking out through emulated devices.
- **[Shared folder and clipboard abuse](shared-folder-and-clipboard-abuse.md)**: the Parallels Tools integration surface.
- **[Known escape exploits](known-escape-exploits.md)**: the recurring device and Tools bugs.

## References

- [Parallels Desktop documentation](https://www.parallels.com/products/desktop/)
- [Zero Day Initiative: Parallels research](https://www.zerodayinitiative.com/blog)
- [Parallels security updates](https://www.parallels.com/products/desktop/)
