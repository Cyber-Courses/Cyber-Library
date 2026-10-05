---
title: "VirtualBox: attacking the Oracle desktop hypervisor"
description: "VirtualBox is a type-2 hypervisor running each VM inside a VBoxSVC/VirtualBox process on the host. Guest-to-host escapes target its emulated devices, 3D acceleration, audio, network, and USB, and the Guest Additions integration (shared folders and clipboard over HGCM). It is a frequent research and Pwn2Own target with a broad device surface."
keywords:
  - virtualbox
  - vboxsvc
  - guest additions
  - hgcm
  - type-2 hypervisor
---

# VirtualBox

VirtualBox is Oracle's cross-platform desktop hypervisor. Each VM runs inside a host process (the `VirtualBox`/`VBoxHeadless` frontend with `VBoxSVC`), which emulates the VM's devices, so a guest-to-host escape corrupts that process and runs on the user's host. Its device surface is broad, 3D acceleration, several audio and network models, and USB controllers, and the Guest Additions add shared folders and clipboard/drag-and-drop over the Host-Guest Communication Manager (HGCM), a prominent integration surface. VirtualBox is a recurring research and Pwn2Own target precisely because of that breadth.

```bash
# from the guest: emulated devices and Guest Additions
lspci -nn; lsusb; lsmod | grep -i vbox
VBoxControl --version 2>/dev/null         # Guest Additions present
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape/index.md)**: breaking out through emulated devices.
- **[Shared folder and clipboard abuse](shared-folder-and-clipboard-abuse.md)**: the Guest Additions HGCM integration.
- **[Known escape exploits](known-escape-exploits.md)**: the recurring device and HGCM bugs.

## References

- [VirtualBox documentation](https://www.virtualbox.org/manual/)
- [Oracle critical patch updates (VirtualBox)](https://www.oracle.com/security-alerts/)
- [Zero Day Initiative: VirtualBox research](https://www.zerodayinitiative.com/blog)
