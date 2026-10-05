---
title: "USB controllers: escaping QEMU through emulated UHCI, EHCI, and XHCI"
description: "QEMU emulates USB host controllers whose guest drivers build transfer descriptors and rings in guest memory that QEMU walks to perform transfers. Flaws in descriptor-chain parsing, in the XHCI ring and device-context model, and in emulated USB device handling give out-of-bounds access in the QEMU process from the guest USB stack."
keywords:
  - usb controller
  - xhci
  - ehci
  - uhci
  - transfer descriptor
---

# USB controllers

QEMU emulates UHCI and EHCI (USB 1.1/2.0) and XHCI (USB 3.0) host controllers, plus a range of emulated USB devices. The guest driver programs a controller through MMIO/PIO registers and builds the controller's data structures, transfer descriptors and queue heads for UHCI/EHCI, or the command, event, and transfer rings of TRBs for XHCI, in guest memory, which QEMU reads to carry out transfers. Walking those guest-built structures, and handling the emulated USB devices behind them, is the escape surface, and the XHCI model in particular has produced memory-safety bugs.

## Driving the controller

```bash
lspci -nn | grep -i usb        # UHCI/EHCI/XHCI model present
# the guest xhci/ehci/uhci driver (or an attacker driver) programs the ring/queue
# base registers and writes descriptors/TRBs with controlled fields
```

```c
// XHCI: the guest sets ring base registers and writes TRBs (type, flags, ptr, len).
// QEMU walks the command/transfer rings following link TRBs and tracks per-slot
// device contexts and endpoints. Primitives:
//  - a link TRB forming a cycle or pointing out of range -> over-read when walked
//  - a transfer length QEMU uses for a copy beyond the mapped buffer
//  - a slot/endpoint index used to index context arrays without bounds -> OOB/UAF
// UHCI/EHCI: frame list + queue-head/transfer-descriptor chains with the same classes
```

## Exploitation notes

- XHCI is the richest target because QEMU tracks substantial per-slot and per-endpoint state, giving type-confusion and use-after-free opportunities on context setup and teardown, in addition to the ring-walk over-reads.
- The emulated USB devices (behind the controller) parse their own guest-influenced protocol and are a secondary surface.
- As with other device models, the guest controls the ring/descriptor base and contents from its driver; exploitation programs the controller directly.
- Primitives land in the QEMU process heap; QEMU-version-specific, pair with a leak. The same controller models appear in other hypervisors, so bugs can cross over.

## References

- [QEMU USB documentation](https://www.qemu.org/docs/master/system/devices/usb.html)
- [QEMU security advisories](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM escape: USB](https://github.com/WinMin/Awesome-VM-Exploit)
