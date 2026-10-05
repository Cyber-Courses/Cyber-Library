---
title: "Guest to host escape: breaking out of an ESXi VM"
description: "Escaping an ESXi guest to the host by exploiting the vmx process device emulation. Each emulated device parses guest input in the host-side process: the SVGA 3D adapter, USB controllers, the virtual NICs, and the backdoor RPC and VMCI channels."
keywords:
  - ESXi escape
  - vmx process
  - SVGA
  - VMCI
  - guest to host
---

# Guest to host escape

Each ESXi VM runs a `vmx` process on the host that emulates its virtual hardware. Every emulated device parses guest-controlled input, so a memory-corruption flaw runs code in the `vmx` process, which is then escalated to the host. The surface splits by device, and because the same device code is shared with [Workstation and Fusion](../../workstation-and-fusion/index.md), escapes often affect all three.

## Subtopics

- **[SVGA and 3D graphics](svga-and-3d-graphics.md)**: the classic ESXi escape surface.
- **[USB controllers](usb-controllers.md)**: UHCI, EHCI, and XHCI emulation.
- **[Virtual NICs](virtual-nics.md)**: the vmxnet3 and e1000 adapters.
- **[Backdoor and VMCI](backdoor-and-vmci.md)**: the guest-to-vmx control channels.

## References

- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
