---
title: "Guest-to-host escape: breaking out of an ESXi virtual machine"
order: 3
description: "Escaping an ESXi guest means corrupting the host-side vmx process that emulates the virtual machine's devices. The reachable surface is the set of emulated devices a guest can drive: the SVGA 3D graphics device, USB controllers, virtual NICs, and the backdoor and VMCI control channels, each parsing guest-controlled data in the host process."
keywords:
  - esxi escape
  - vmx process
  - device emulation
  - guest-to-host
  - hypervisor escape
---

# Guest-to-host escape

An ESXi virtual machine runs under a per-VM `vmx` user-space process on the host that emulates the VM's devices, backed by the `vmkernel` hypervisor. A guest escapes by making `vmx` mishandle data the guest controls, so the target is the device-emulation code: the SVGA graphics device with its 3D command stream, the emulated USB controllers, the virtual network adapters, and the backdoor and VMCI control channels. Each reads guest-supplied structures (command FIFOs, transfer descriptors, ring buffers, RPC messages) in the host process, and a memory-safety flaw there yields code execution in `vmx`, which runs with enough privilege to reach the host.

The escape surface a guest can reach from inside:

```bash
# enumerate the emulated hardware the VM exposes (from a Linux guest)
lspci -nn                       # SVGA II, vmxnet3/e1000, USB (UHCI/EHCI/XHCI), VMCI
lsusb; ls /sys/bus/pci/devices  # USB controllers and PCI functions
dmesg | grep -iE 'vmwgfx|vmxnet|vmw_vmci'   # loaded VMware guest drivers
```

## Subtopics

- **[Backdoor and VMCI](backdoor-and-vmci.md)**: the control channels into the vmx process.
- **[SVGA and 3D graphics](svga-and-3d-graphics.md)**: the graphics device command stream.
- **[USB controllers](usb-controllers.md)**: emulated UHCI, EHCI, and XHCI.
- **[Virtual NICs](virtual-nics.md)**: the e1000 and vmxnet3 network adapters.

## References

- [Zero Day Initiative: VMware escape research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
- [Phrack and conference talks on VMware vmx escapes](https://www.zerodayinitiative.com/blog)
