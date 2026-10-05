---
title: "Guest to host escape: breaking out of an ESXi VM"
description: "Escaping an ESXi guest to the host by exploiting the vmx process device emulation: the SVGA 3D graphics adapter, USB and network controllers, and the backdoor RPC channel, which parse guest-controlled input in the host-side process and have repeatedly yielded host code execution."
keywords:
  - ESXi escape
  - vmx process
  - SVGA
  - VMCI
  - guest to host
---

# Guest to host escape

Each ESXi VM runs a `vmx` process on the host that emulates the guest's virtual hardware. That process parses guest-controlled input for every emulated device, so memory-corruption flaws in the SVGA 3D adapter, USB and network controllers, or the backdoor RPC and VMCI channels let a guest run code in the `vmx` process, which is then escalated to the host.

```text
High-value ESXi escape surfaces (reachable from a guest):
- SVGA / 3D graphics (vmware-vmx), the classic escape surface
- USB controllers (UHCI/EHCI/XHCI)
- Virtual NICs (vmxnet3, e1000)
- The backdoor RPC channel and VMCI
```

## Exploitation notes

- The SVGA 3D path is the most productive historically; disabling 3D acceleration removes it, so its reachability depends on the guest's video configuration.
- A `vmx` compromise is host user code in the VM's process; escalation to full host follows, and several public chains do both.
- Named, weaponized instances are under [Known escape exploits](known-escape-exploits.md); the desktop products share this code, see [Workstation and Fusion](../workstation-and-fusion/index.md).

## References

- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
