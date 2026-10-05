---
title: "USB controllers: escaping ESXi through emulated UHCI, EHCI, and XHCI"
description: "ESXi emulates USB host controllers (UHCI, EHCI, XHCI) whose guest drivers build transfer and queue descriptors in guest memory that the vmx process reads and walks via DMA. Flaws in descriptor-chain parsing, ring handling, or device-request processing give out-of-bounds access in the host process, a guest-to-host escape driven entirely from the guest USB stack."
keywords:
  - usb controller
  - xhci
  - ehci
  - uhci
  - transfer descriptor
---

# USB controllers

A guest sees one or more emulated USB host controllers: UHCI and EHCI for USB 1.1/2.0, and XHCI for USB 3.0. The guest driver programs each controller through MMIO or I/O registers and builds data structures in guest memory, transfer descriptors and queue heads for UHCI/EHCI, and the command, event, and transfer rings with their TRBs for XHCI, that the host-side controller model in `vmx` reads by DMA and walks. Parsing those guest-built structures is the escape surface: a descriptor chain or ring that the host walks with a trusted length or a dangling link yields out-of-bounds access in `vmx`.

## Driving the controller from the guest

```bash
# identify the emulated controllers
lspci -nn | grep -i usb        # Intel UHCI/EHCI, or an XHCI controller
# the guest xhci/ehci driver programs the controller; from an attacker driver you
# control the register programming and the descriptor/ring contents directly
```

For XHCI the key guest-built structures are the command ring, the event ring, and per-endpoint transfer rings, each a sequence of Transfer Request Blocks (TRBs) with a type, flags, and a data pointer. The host model reads the ring base from a register the guest sets, then walks TRBs following link TRBs:

```c
// a malicious guest points a ring base register at attacker memory and writes
// TRBs with: an oversized transfer length, a link TRB forming a cycle or pointing
// out of range, or a device-context/endpoint index the host indexes without bounds
// checking -> OOB read/write when vmx walks the ring or copies the transfer
```

For UHCI/EHCI the equivalent is the frame list and queue-head/transfer-descriptor chains, where a crafted chain (loops, out-of-range pointers, inconsistent lengths) drives the host walker out of bounds.

## Exploitation notes

- The controller model trusts the ring/descriptor base and link pointers the guest programs; a link forming a cycle or pointing outside guest RAM, or a TRB length the host uses for a copy, is the typical primitive.
- XHCI is the modern high-value target because its ring and device-context model is complex and the host must track significant per-endpoint state, giving type-confusion and use-after-free opportunities on endpoint and context teardown.
- The emulated-device USB descriptors (returned to the guest) are a separate surface for host-to-guest influence, but the escape direction here is guest-built rings read by the host.
- As with the other devices, a build-specific bug plus an ASLR leak is needed; fingerprint the ESXi version.

## References

- [xHCI specification](https://www.intel.com/content/dam/www/public/us/en/documents/technical-specifications/extensible-host-controller-interface-usb-xhci.pdf)
- [Zero Day Initiative: VMware USB research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
