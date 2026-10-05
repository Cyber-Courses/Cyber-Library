---
title: "Guest to host escape: breaking out of VirtualBox"
description: "Escaping a VirtualBox guest to the host through its emulated devices: the e1000 and PCNet network adapters, the VGA and 3D graphics, and USB and audio controllers, which run in the host VBoxSVC and VM processes and parse guest-controlled input."
keywords:
  - VirtualBox escape
  - e1000
  - 3D acceleration
  - device emulation
  - guest to host
---

# Guest to host escape

VirtualBox emulates the guest's hardware in host-side processes. Every emulated device parses guest-controlled input, so memory-corruption flaws in the network adapters (e1000, PCNet), the VGA and 3D graphics, and the USB and audio controllers let a guest execute code on the host. VirtualBox's broad, long-standing device set makes this a rich surface.

```text
High-value VirtualBox escape surfaces (reachable from a guest):
- Network adapters: e1000, PCNet
- Graphics: VGA / VBoxVGA and the 3D (Chromium/VMSVGA) path
- USB controllers (OHCI/EHCI/XHCI)
- Audio (AC97/HDA)
```

## Exploitation notes

- The e1000 network adapter and the 3D graphics path are recurring, productive targets; 3D acceleration is off by default, so its reachability depends on the guest's configuration.
- Escapes land in the host process running the VM as the user who started it, then escalate.
- Named instances are under [Known escape exploits](known-escape-exploits.md).

## References

- [Oracle VirtualBox manual](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox research](https://www.zerodayinitiative.com/blog)
