---
title: "Guest-to-host escape: breaking out of a QEMU/KVM virtual machine"
description: "A QEMU guest escapes by corrupting the QEMU process on the host, which emulates the VM's devices while KVM accelerates the CPU. The reachable surface is QEMU's device models: virtio devices, emulated NICs, USB controllers, block and SCSI controllers, audio devices, and the legacy floppy controller, each parsing guest-driven register writes and DMA descriptors."
keywords:
  - qemu escape
  - kvm
  - device model
  - virtio
  - guest-to-host
---

# Guest-to-host escape

QEMU provides the device emulation for a KVM virtual machine while the KVM kernel module accelerates CPU and memory virtualization. A guest cannot touch the host CPU path, so escapes target the QEMU user-space process: its device models read guest-driven I/O (port and MMIO register writes) and walk DMA descriptors and ring buffers the guest places in its own memory. A memory-safety flaw in any device model yields code execution in the QEMU process, which runs on the host with the privileges of whoever launched the VM (often reduced by seccomp and sometimes confined, but still a host process). Because the device models are shared, these bugs recur across every KVM-based product.

```bash
# from the guest: enumerate the emulated devices (the escape surface)
lspci -nn                       # virtio, e1000/rtl8139, USB controllers, audio
lsusb; cat /proc/ioports        # I/O-port devices incl. the legacy floppy (0x3f0-)
dmesg | grep -i virtio
```

## Subtopics

- **[virtio devices](virtio-devices.md)**: the paravirtual virtqueue device family.
- **[Network adapters](network-adapters.md)**: emulated e1000, rtl8139, and virtio-net.
- **[USB controllers](usb-controllers.md)**: emulated UHCI, EHCI, and XHCI.
- **[Block and SCSI](block-and-scsi.md)**: the IDE/AHCI and virtio-blk/SCSI storage paths.
- **[Audio devices](audio-devices.md)**: the emulated sound cards.
- **[Floppy controller](floppy-controller.md)**: the legacy floppy disk controller.

## References

- [QEMU documentation](https://www.qemu.org/docs/master/)
- [QEMU security process and advisories](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM/hypervisor escape research](https://github.com/WinMin/Awesome-VM-Exploit)
