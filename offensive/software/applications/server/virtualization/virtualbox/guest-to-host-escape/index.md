---
title: "Guest to host escape: breaking out of VirtualBox"
description: "Escaping a VirtualBox guest to the host through its emulated devices, each running in a host-side process and parsing guest-controlled input: the network adapters, the 3D acceleration path, the USB controllers, and the audio devices."
keywords:
  - VirtualBox escape
  - e1000
  - 3D acceleration
  - device emulation
  - guest to host
---

# Guest to host escape

VirtualBox emulates the guest's hardware in host-side processes. Every emulated device parses guest-controlled input, so a memory-corruption flaw runs code on the host as the user running the VM. VirtualBox's broad, open-source device set makes this a well-studied surface, with public exploit chains for several devices. The surface splits by device.

## Subtopics

- **[Network adapters](network-adapters.md)**: the e1000 and PCNet models.
- **[3D acceleration](3d-acceleration.md)**: the Chromium and VMSVGA graphics path.
- **[USB controllers](usb-controllers.md)**: OHCI, EHCI, and XHCI emulation.
- **[Audio devices](audio-devices.md)**: AC97, Intel HDA, and SB16.

## References

- [Oracle VirtualBox manual](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox research](https://www.zerodayinitiative.com/blog)
