---
title: "Guest-to-host escape: breaking out of a VirtualBox virtual machine"
description: "A VirtualBox guest escapes by corrupting the host VM process that emulates its devices. The reachable surface is the emulated device set: the 3D acceleration path, the audio controllers, the network adapters, and the USB controllers, each parsing guest-driven register writes and DMA structures, plus the HGCM services behind Guest Additions."
keywords:
  - virtualbox escape
  - device emulation
  - 3d acceleration
  - usb
  - guest-to-host
---

# Guest-to-host escape

A VirtualBox VM runs inside a host process that emulates its devices, so escaping means making that process mishandle guest-controlled data. The device surface is broad: the 3D acceleration path (used by Guest Additions for accelerated graphics), the emulated audio controllers, the network adapters, and the USB controllers, each reading guest register writes and DMA descriptors on the host. Code execution lands in the VM process, which runs with the privileges of the user who started the VM. The Guest Additions integration (shared folders, clipboard) adds the HGCM service surface, covered separately.

```bash
# the emulated devices visible in the guest (escape surface)
lspci -nn; lsusb
dmesg | grep -iE 'e1000|pcnet|ac97|hda|ohci|ehci|xhci'
```

## Subtopics

- **[3D acceleration](3d-acceleration.md)**: the accelerated-graphics command path.
- **[Audio devices](audio-devices.md)**: the emulated AC97 and HD Audio controllers.
- **[Network adapters](network-adapters.md)**: the e1000, PCnet, and virtio-net models.
- **[USB controllers](usb-controllers.md)**: the OHCI, EHCI, and xHCI controllers.

## References

- [VirtualBox manual: virtual hardware](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox escapes](https://www.zerodayinitiative.com/blog)
- [Oracle security alerts](https://www.oracle.com/security-alerts/)
