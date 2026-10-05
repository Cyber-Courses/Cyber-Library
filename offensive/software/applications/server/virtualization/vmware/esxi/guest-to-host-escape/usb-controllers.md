---
title: "USB controllers: escaping ESXi through USB emulation"
description: "Escaping ESXi through the emulated USB controllers (UHCI, EHCI, XHCI) in the vmx process, which parse guest-issued transfer descriptors and device requests, reachable when a USB controller is present on the guest."
keywords:
  - USB controller
  - XHCI
  - vmx
  - ESXi escape
  - device emulation
---

# USB controllers

ESXi emulates USB host controllers (UHCI, EHCI, XHCI) in the `vmx` process. They process the guest's transfer descriptors and transfer rings, and the emulated USB devices handle guest requests. Flaws in the controller or device handling corrupt memory in the host-side process, a recurring secondary escape surface after graphics.

```text
USB escape surface:
- Host controllers: UHCI, EHCI, XHCI (transfer ring / descriptor parsing)
- Emulated USB device models
```

## Exploitation notes

- The XHCI controller has the richest command and transfer-ring surface and has produced escapes.
- Reachability requires a USB controller on the guest, which is common; the device code is shared with Workstation and Fusion.
- The handling runs in the `vmx` process, then escalates to the host.

## References

- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
