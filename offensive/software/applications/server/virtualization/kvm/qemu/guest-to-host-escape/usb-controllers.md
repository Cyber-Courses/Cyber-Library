---
title: "USB controllers: escaping through QEMU USB emulation"
description: "Escaping a KVM guest through the QEMU USB controller emulation (UHCI, EHCI, XHCI) and the emulated USB device models, which parse guest-issued transfer descriptors and device requests in the host QEMU process."
keywords:
  - USB controller
  - UHCI
  - EHCI
  - XHCI
  - QEMU escape
---

# USB controllers

QEMU emulates USB host controllers (UHCI, EHCI, XHCI) and a set of USB devices. The controllers process the guest's transfer descriptors and schedules, and the device models handle guest requests, all in the host QEMU process. Flaws in the controller schedule processing or in a device model corrupt host memory.

```text
USB escape surface:
- Host controllers: UHCI, EHCI, XHCI (transfer descriptor / schedule parsing)
- Emulated device models (HID, storage, serial)
```

## Exploitation notes

- The XHCI controller, with its richer command and transfer-ring model, is a larger surface than the legacy UHCI and EHCI.
- Reachability requires a USB controller on the guest, which is common; adding a USB device widens the device-model surface.
- The handling runs in the QEMU process, bounded by the host's confinement.

## References

- [QEMU USB emulation](https://www.qemu.org/docs/master/system/devices/usb.html)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
