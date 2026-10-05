---
title: "KVM and QEMU: attacking Linux virtualization"
description: "Attacking KVM-based virtualization on Linux: reaching the host and the libvirt control socket, escaping a guest to the host through QEMU device-model flaws, abusing the libvirt and oVirt management plane, and stealing qcow2 disks and snapshots. KVM underpins Proxmox, Nutanix, OpenStack, and most VPS providers."
keywords:
  - KVM
  - QEMU
  - libvirt
  - VM escape
  - qcow2
---

# KVM and QEMU

KVM is the Linux kernel's hypervisor, and QEMU provides the device emulation for its guests. The pairing underpins Proxmox, Nutanix AHV, OpenStack, and most VPS providers, so KVM and QEMU techniques generalize across the Linux virtualization world. The targets are the host and its libvirt control, the QEMU device models that back guest-to-host escapes, the management plane, and the qcow2 disks at rest.

## Subtopics

- **[Host access and shell](host-access-and-shell.md)**: reaching the host and libvirt.
- **[Guest to host escape](guest-to-host-escape.md)**: breaking out through QEMU device models.
- **[Management plane](management-plane.md)**: libvirt, oVirt, and RHV.
- **[Disk and snapshot theft](disk-and-snapshot-theft.md)**: qcow2 and raw images.
- **[Known escape exploits](known-escape-exploits.md)**: named QEMU and KVM breakouts.

## References

- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
- [libvirt documentation](https://libvirt.org/docs.html)
