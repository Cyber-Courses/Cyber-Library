---
title: "QEMU: attacking the base KVM virtualization stack"
description: "Attacking KVM-based virtualization on Linux: reaching the host and the libvirt control socket, escaping a guest to the host through QEMU device-model flaws, abusing the libvirt and oVirt management plane, and stealing qcow2 disks and snapshots. KVM underpins Proxmox, Nutanix, OpenStack, and most VPS providers."
keywords:
  - KVM
  - QEMU
  - libvirt
  - VM escape
  - qcow2
---

# QEMU

QEMU is the user-space virtual machine monitor that provides device emulation for KVM guests, and it is the base of the Linux virtualization stack that Proxmox, Nutanix AHV, and OpenStack build on. The targets here are the host and its libvirt control, the QEMU device models that back guest-to-host escapes, the libvirt management plane, and the qcow2 disks at rest. The KVM kernel accelerator beneath it is a separate target.

## Subtopics

- **[Host access and shell](host-access-and-shell.md)**: reaching the host and libvirt.
- **[Guest to host escape](guest-to-host-escape.md)**: breaking out through QEMU device models.
- **[Management plane](management-plane.md)**: libvirt, oVirt, and RHV.
- **[Disk and snapshot theft](disk-and-snapshot-theft.md)**: qcow2 and raw images.
- **[Known escape exploits](known-escape-exploits.md)**: named QEMU device breakouts.

## References

- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
- [libvirt documentation](https://libvirt.org/docs.html)
