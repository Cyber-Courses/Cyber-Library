---
title: "Guest to host escape: breaking out of Workstation or Fusion"
description: "Escaping a VMware Workstation or Fusion guest to the host through the vmware-vmx device-emulation code shared with ESXi: the SVGA 3D adapter, USB controllers, and virtual NICs, which run in a host process and parse guest-controlled input."
keywords:
  - Workstation escape
  - Fusion escape
  - vmware-vmx
  - SVGA
  - guest to host
---

# Guest to host escape

Workstation and Fusion run each VM through a `vmware-vmx` process on the host desktop, the same device-emulation code as ESXi. The escape surface is therefore the same: the SVGA 3D graphics adapter, USB controllers, and virtual NICs parse guest input in the host process, and memory-corruption flaws there run code on the host.

```text
Shared escape surface with ESXi:
- SVGA / 3D graphics (the primary target)
- USB controllers (UHCI/EHCI/XHCI)
- Virtual NICs (vmxnet3, e1000)
```

## Exploitation notes

- Because the device code is shared, escapes often affect ESXi, Workstation, and Fusion together; the difference is the host OS the escape lands on.
- Analysis sandboxes commonly leave 3D acceleration and Shared Folders enabled, widening the surface; see [Shared folder and drag-drop abuse](shared-folder-and-drag-drop-abuse.md).
- Named instances are under [Known escape exploits](known-escape-exploits.md); the ESXi view is [ESXi guest to host escape](../esxi/guest-to-host-escape/index.md).

## References

- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
