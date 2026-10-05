---
title: "Workstation and Fusion: attacking the VMware desktop hypervisors"
description: "VMware Workstation (Windows/Linux) and Fusion (macOS) are type-2 hypervisors that run VMs as a vmware-vmx process on the user's host. They share ESXi's device-emulation and GuestRPC code, so the guest-to-host escapes target the same devices, and the shared-folder and drag-and-drop integration features are a prominent escape surface."
keywords:
  - vmware workstation
  - vmware fusion
  - vmware-vmx
  - type-2 hypervisor
  - shared folders
---

# Workstation and Fusion

Workstation (Windows and Linux) and Fusion (macOS) are VMware's hosted hypervisors: each VM runs as a `vmware-vmx` process in the user's session, emulating the same devices as ESXi. A guest escape here corrupts that process and then runs on the user's host. Because the device-emulation and GuestRPC code is shared with ESXi, the escape surface is the same, SVGA 3D, USB, NICs, and the backdoor/VMCI channels, and the desktop integration features (shared folders via HGFS, drag-and-drop, and copy-paste) add a prominent, frequently vulnerable surface specific to how these products bridge guest and host.

```bash
# from the guest: same emulated devices as ESXi
lspci -nn; lsusb
# host side: the per-VM process is vmware-vmx (Windows) / vmware-vmx (macOS/Linux)
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape.md)**: breaking out of a VM to the user's host.
- **[Shared folder and drag-drop abuse](shared-folder-and-drag-drop-abuse.md)**: the HGFS and clipboard integration surface.
- **[Known escape exploits](known-escape-exploits.md)**: the recurring desktop escape bugs.

## References

- [VMware Workstation documentation](https://docs.vmware.com/en/VMware-Workstation-Pro/index.html)
- [Zero Day Initiative: VMware desktop escapes](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
