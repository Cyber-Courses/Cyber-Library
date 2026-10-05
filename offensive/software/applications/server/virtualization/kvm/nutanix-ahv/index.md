---
title: "Nutanix AHV: attacking the HCI hypervisor and Prism"
description: "Attacking Nutanix AHV, the KVM-based hypervisor in Nutanix hyperconverged infrastructure: reaching the AHV host and the Controller VM that fronts storage, escaping a guest through the QEMU stack, abusing the Prism management plane, and stealing disks from the distributed storage fabric."
keywords:
  - Nutanix AHV
  - Prism
  - Controller VM
  - HCI
  - VM escape
---

# Nutanix AHV

Nutanix AHV is a KVM-based type-1 hypervisor at the core of Nutanix hyperconverged infrastructure, and it has absorbed much of the migration away from VMware. Each node runs AHV plus a Controller VM (CVM) that provides the distributed storage fabric, and the whole cluster is managed through Prism. Offensively it is KVM underneath, with Nutanix-specific host, CVM, storage, and Prism surfaces on top.

## Subtopics

- **[Host access and shell](host-access-and-shell.md)**: reaching the AHV host and CVM.
- **[Guest to host escape](guest-to-host-escape.md)**: breaking out through QEMU.
- **[Management plane](management-plane.md)**: Prism and its API.
- **[Disk and snapshot theft](disk-and-snapshot-theft.md)**: the storage fabric.
- **[Known escape exploits](known-escape-exploits.md)**: Nutanix, QEMU, and Prism flaws.

## References

- [Nutanix AHV administration](https://portal.nutanix.com/page/documents/list?type=software&filterKey=software&filterVal=AHV)
- [Nutanix security advisories](https://www.nutanix.com/support-services/security-advisories)
