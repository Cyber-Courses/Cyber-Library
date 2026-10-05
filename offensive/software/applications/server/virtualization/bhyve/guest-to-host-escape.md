---
title: "Guest-to-host escape: breaking out of a bhyve virtual machine"
description: "A bhyve guest escapes by corrupting the userspace bhyve process that emulates its devices. The reachable surface is bhyve's device models, the virtio family, the AHCI storage controller, the e1000 NIC, the USB controllers, and the framebuffer, each parsing guest-driven register writes and DMA descriptors, with a flaw yielding code execution in the bhyve process on the FreeBSD host."
keywords:
  - bhyve escape
  - virtio
  - ahci
  - e1000
  - device model
---

# Guest-to-host escape

bhyve emulates a VM's devices in a per-VM userspace process, with the `vmm.ko` kernel module accelerating the CPU. Escaping a bhyve guest means making that `bhyve` process mishandle guest-controlled data in a device model. The surface is bhyve's device set: the virtio devices (virtio-blk, virtio-net, virtio-9p, virtio-console), the AHCI SATA controller, the e1000 NIC, the USB (XHCI) controller, and the framebuffer. Each reads guest register writes and DMA descriptors, so a memory-safety flaw gives code execution in the `bhyve` process on the FreeBSD host.

```bash
# the emulated devices visible in the guest (escape surface)
pciconf -lv 2>/dev/null | grep -iE 'virtio|ahci|e1000|xhci'   # (FreeBSD guest)
lspci -nn 2>/dev/null | grep -iE 'virtio|ahci|intel|usb'      # (Linux guest)
```

## The device surface

```c
// bhyve device models parse guest-controlled structures:
//  - virtio (blk/net/9p/console): virtqueue descriptors (addr/len/flags), indirect
//    descriptors, and per-device headers; a trusted length/index -> OOB in bhyve
//  - AHCI: command list + PRD tables with guest addresses/counts for DMA
//  - e1000: TX/RX descriptors and offload fields
//  - XHCI: command/transfer rings of TRBs with link/length fields
// the mechanisms match the equivalent QEMU device models
```

The bug classes are the standard device-emulation ones, a length or count the model trusts, an index it does not bound, a use-after-free on a request object, and because bhyve emphasises virtio, the virtqueue handling (shared mechanism, see [QEMU virtio devices](../kvm/qemu/guest-to-host-escape/virtio-devices.md)) is a primary surface.

## Exploitation notes

- Code execution lands in the userspace `bhyve` process on the FreeBSD host; it runs with the privileges of whoever launched the VM, so a further local privilege escalation may be needed for full host control, as with other type-2 setups.
- bhyve's lean, virtio-focused design means the virtio and AHCI paths are the primary surfaces; the mechanisms mirror QEMU's, so that device analysis transfers.
- The guest drives the device models from a controlled driver, posting crafted descriptors directly; primitives land in the bhyve heap and are version-specific.
- FreeBSD security advisories track bhyve device bugs; fingerprint the FreeBSD/bhyve version.

## References

- [bhyve(8) and device models](https://man.freebsd.org/cgi/man.cgi?query=bhyve)
- [FreeBSD security advisories (bhyve)](https://www.freebsd.org/security/advisories/)
