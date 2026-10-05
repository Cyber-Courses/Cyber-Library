---
title: "USB controllers: escaping VirtualBox through USB emulation"
description: "Escaping a VirtualBox guest through its USB controller emulation (OHCI, EHCI, XHCI), which parses guest-issued transfer descriptors and device requests in the host-side VM process, reachable when a USB controller is attached to the guest."
keywords:
  - USB controller
  - OHCI
  - XHCI
  - VirtualBox escape
  - device emulation
---

# USB controllers

VirtualBox emulates OHCI, EHCI, and XHCI USB host controllers in the host-side VM process. They process the guest's transfer descriptors and schedules, and the emulated devices handle guest requests. Flaws in the controller or device handling corrupt host memory, a recurring VirtualBox escape surface.

```text
USB escape surface:
- Host controllers: OHCI, EHCI, XHCI (transfer descriptor / schedule parsing)
- Emulated USB device models
```

## Exploitation notes

- The XHCI controller has the richest surface; reachability requires a USB controller on the guest, which is commonly present.
- VirtualBox's open-source device code makes these controllers heavily audited, with public findings.
- Code execution lands in the host VM process, then escalates.

## References

- [Oracle VirtualBox manual](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox research](https://www.zerodayinitiative.com/blog)
