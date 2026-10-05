---
title: "Guest to host escape: breaking out of a KVM guest through QEMU"
description: "Escaping a KVM guest to the host by exploiting QEMU device-model flaws: the virtio devices, the e1000 and rtl8139 network adapters, USB and SCSI controllers, and the legacy floppy controller, which run in the host QEMU process and parse guest-controlled input."
keywords:
  - QEMU escape
  - virtio
  - device model
  - VENOM
  - guest to host
---

# Guest to host escape

Each KVM guest is served by a QEMU process on the host that emulates its devices in user space. Those device models parse guest-controlled input, so memory-corruption flaws in them run code in the host QEMU process. The surface spans virtio devices, the legacy network adapters (e1000, rtl8139), USB and SCSI controllers, and the floppy controller behind the VENOM class of escape.

```text
High-value QEMU escape surfaces (reachable from a guest):
- virtio devices (net, block, scsi, gpu) over the virtqueue
- Legacy NICs: e1000, rtl8139
- USB (UHCI/EHCI/XHCI) and SCSI controllers
- The floppy disk controller (the VENOM surface)
```

## Exploitation notes

- Code execution lands in the host QEMU process, whose privileges depend on hardening: a QEMU confined by seccomp, SELinux/AppArmor (sVirt), and a non-root user limits the blast radius, so check the host's confinement.
- virtio is the broadest modern surface because every paravirtualized guest uses it; legacy devices depend on the machine type.
- Named instances, including VENOM, are under [Known escape exploits](known-escape-exploits.md); the same QEMU code backs [Proxmox](../proxmox-ve/guest-to-host-escape.md) and [Nutanix AHV](../nutanix-ahv/guest-to-host-escape.md).

## References

- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
- [libvirt: sVirt confinement](https://libvirt.org/drvqemu.html#security-driver)
