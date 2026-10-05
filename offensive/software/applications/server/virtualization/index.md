---
title: "Virtualization: attacking hypervisors and their management planes"
description: "Attacking virtualization platforms: escaping a guest VM to the host through hypervisor and device-emulation flaws, taking over the management plane that controls many hosts, and stealing virtual disks and snapshots offline, across VMware, Hyper-V, KVM, Xen, Proxmox, Nutanix, and more."
keywords:
  - virtualization security
  - hypervisor
  - VM escape
  - guest to host escape
  - management plane
---

# Virtualization

A hypervisor runs untrusted guest operating systems and is supposed to keep them isolated from the host and from each other. Offensive work targets the places where that isolation leaks: the device models the hypervisor emulates for guests (the usual guest-to-host escape surface), the management plane that controls fleets of hosts, and the virtual disks and snapshots that hold guest data at rest.

The area is organized by product, because escapes are specific to each hypervisor's device-emulation code, with a consistent set of aspects under each.

## Subtopics

- **[VMware](vmware/index.md)**: ESXi and vCenter, plus the Workstation and Fusion desktop hypervisors.
- **[Hyper-V](hyper-v/index.md)**: the Windows hypervisor and its management stack.
- **[VirtualBox](virtualbox/index.md)**: the Oracle desktop hypervisor.
- **[KVM and QEMU](kvm-and-qemu/index.md)**: Linux virtualization and the QEMU device models.
- **[Xen](xen/index.md)**: the Xen hypervisor and its control domain.
- **[Proxmox VE](proxmox-ve/index.md)**: the Debian, KVM, and LXC platform.
- **[Nutanix AHV](nutanix-ahv/index.md)**: the KVM-based hypervisor in Nutanix HCI.
- **[Parallels Desktop](parallels-desktop/index.md)**: the macOS desktop hypervisor.
- **[bhyve](bhyve/index.md)**: the FreeBSD hypervisor.
- **[microVM monitors](microvm-monitors/index.md)**: Firecracker and Cloud Hypervisor.

## References

- [Pwn2Own results archive](https://www.zerodayinitiative.com/blog)
- [QEMU security process](https://www.qemu.org/docs/master/system/security.html)
