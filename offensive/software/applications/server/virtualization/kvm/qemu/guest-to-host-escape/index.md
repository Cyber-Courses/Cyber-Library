---
title: "Guest to host escape: breaking out of a KVM guest through QEMU"
description: "Escaping a KVM guest to the host by exploiting QEMU device-model flaws. Each emulated device runs in the host QEMU process and parses guest-controlled input: virtio, the legacy network adapters, USB and storage controllers, the floppy controller, and audio."
keywords:
  - QEMU escape
  - device model
  - virtio
  - VENOM
  - guest to host
---

# Guest to host escape

Each KVM guest is served by a QEMU process on the host that emulates its devices in user space. Those device models parse guest-controlled input, so a memory-corruption flaw in one runs code in the host QEMU process. The surface splits by device, and which models are reachable depends on the guest's configured hardware.

Code execution lands in the host QEMU process, whose reach depends on the host's confinement (seccomp, sVirt, non-root QEMU). A flaw in the [KVM kernel module](../../kernel-module/kernel-module-escape.md) beneath QEMU bypasses that confinement entirely. The same QEMU code backs [Proxmox](../../proxmox-ve/guest-to-host-escape.md) and [Nutanix AHV](../../nutanix-ahv/guest-to-host-escape.md).

## Subtopics

- **[virtio devices](virtio-devices.md)**: the virtqueue-based paravirtualized devices.
- **[Network adapters](network-adapters.md)**: the e1000 and rtl8139 models.
- **[USB controllers](usb-controllers.md)**: UHCI, EHCI, and XHCI emulation.
- **[Block and SCSI](block-and-scsi.md)**: AHCI, IDE, and emulated SCSI.
- **[Floppy controller](floppy-controller.md)**: the VENOM class.
- **[Audio devices](audio-devices.md)**: AC97, Intel HDA, and ES1370.

## References

- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
- [QEMU device emulation](https://www.qemu.org/docs/master/system/devices.html)
