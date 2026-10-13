---
title: "USB controllers: escaping VirtualBox through emulated OHCI, EHCI, and xHCI"
order: 3
description: "VirtualBox emulates OHCI, EHCI, and xHCI USB host controllers whose guest drivers build transfer descriptors and rings in guest memory that the host VM process walks via DMA. Flaws in descriptor-chain and xHCI ring and device-context handling give out-of-bounds access in the host process, a recurring VirtualBox escape surface including at Pwn2Own."
keywords:
  - usb controller
  - xhci
  - ehci
  - ohci
  - transfer descriptor
---

# USB controllers

VirtualBox emulates OHCI (USB 1.1), EHCI (USB 2.0), and xHCI (USB 3.0) host controllers. The guest driver programs a controller through registers and builds its data structures, endpoint and transfer descriptors for OHCI/EHCI, and the command, event, and transfer rings of TRBs for xHCI, in guest memory, which the host VM process reads by DMA to perform transfers. Walking those guest-built structures is the escape surface, and the xHCI model in particular has been a VirtualBox escape vector, including at Pwn2Own.

## The surface

```c
// xHCI: the guest sets ring base registers and writes TRBs (type, flags, ptr, len).
// the host walks the command/transfer rings following link TRBs and tracks per-slot
// device contexts and endpoints. Primitives:
//  - a link TRB forming a cycle or pointing out of range -> over-read when walked
//  - a transfer length the host uses for a copy beyond the mapped buffer
//  - a slot/endpoint index used to index context arrays without bounds -> OOB/UAF
// OHCI/EHCI: endpoint/transfer descriptor lists with the same descriptor-walk classes
```

```bash
lspci -nn | grep -i usb       # OHCI/EHCI/xHCI present (xHCI needs the extension pack era config)
# an attacker guest USB driver programs the rings/descriptors directly
```

## Exploitation notes

- xHCI is the richest target because the host tracks substantial per-slot and per-endpoint state, giving type-confusion and use-after-free opportunities on context setup/teardown in addition to ring-walk over-reads.
- The guest controls the ring/descriptor base and contents from its driver; exploitation programs the controller directly rather than attaching a device normally.
- The same controller models appear in QEMU and VMware, so xHCI bugs recur across hypervisors; see [QEMU USB controllers](../../kvm/qemu/guest-to-host-escape/usb-controllers.md).
- Primitives land in the VM process; version-specific, pair with a leak.

## References

- [xHCI specification](https://www.intel.com/content/dam/www/public/us/en/documents/technical-specifications/extensible-host-controller-interface-usb-xhci.pdf)
- [Zero Day Initiative: VirtualBox USB research](https://www.zerodayinitiative.com/blog)
- [Oracle security alerts](https://www.oracle.com/security-alerts/)
