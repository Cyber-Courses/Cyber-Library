---
title: "KVM: attacking the Linux KVM virtualization ecosystem"
description: "Attacking the KVM virtualization ecosystem on Linux: the QEMU VMM and its device models, the KVM kernel acceleration module, and the platforms and monitors built on them, Proxmox VE, Nutanix AHV, and the Firecracker and Cloud Hypervisor microVM monitors."
keywords:
  - KVM
  - QEMU
  - Proxmox
  - Nutanix
  - virtualization
---

# KVM

KVM is the Linux kernel's hardware-acceleration layer for virtualization, and it underpins most of the Linux virtualization world. A VMM in user space (usually QEMU, sometimes a minimal monitor) provides the device emulation, while the KVM kernel module provides the CPU and memory acceleration. The same core powers Proxmox, Nutanix AHV, OpenStack, and the serverless microVM monitors, so this grouping collects every KVM-based platform.

## Subtopics

- **[QEMU](qemu/index.md)**: the base VMM, its device-model escapes, and the libvirt host and management.
- **[Kernel module](kernel-module/index.md)**: the KVM accelerator itself and nested virtualization.
- **[Proxmox VE](proxmox-ve/index.md)**: the Debian, KVM, and LXC platform.
- **[Nutanix AHV](nutanix-ahv/index.md)**: the KVM-based hypervisor in Nutanix HCI.
- **[microVM monitors](microvm-monitors/index.md)**: Firecracker and Cloud Hypervisor.

## References

- [KVM documentation](https://www.linux-kvm.org/page/Documents)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
