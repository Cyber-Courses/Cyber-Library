---
title: "Xen: attacking the Xen hypervisor and its control domain"
description: "Attacking Xen virtualization: reaching the privileged control domain (dom0), escaping a guest to the hypervisor through hypercalls, grant tables, and backend drivers or QEMU device models, abusing the XCP-ng and XenServer management plane, and stealing guest virtual disks."
keywords:
  - Xen
  - dom0
  - hypercall
  - XCP-ng
  - VM escape
---

# Xen

Xen is a type-1 hypervisor with a privileged control domain, dom0, that manages the unprivileged guest domains (domU). Offensive targets are dom0 itself, the guest-to-host escape surface (hypercalls, grant tables, paravirtualized backend drivers, and the QEMU device models used for HVM guests), the XCP-ng and XenServer management plane, and the virtual disks on the storage repository. Xen powers cloud platforms and Citrix products.

## Subtopics

- **[Host access and shell](host-access-and-shell.md)**: reaching dom0.
- **[Guest to host escape](guest-to-host-escape/index.md)**: breaking out to dom0 or the hypervisor.
- **[Management plane](management-plane.md)**: XCP-ng, XenServer, and xapi.
- **[Disk and snapshot theft](disk-and-snapshot-theft.md)**: reading guest disks.
- **[Known escape exploits](known-escape-exploits.md)**: named Xen breakouts.

## References

- [Xen Project documentation](https://xenproject.org/help/documentation/)
- [Xen security advisories](https://xenbits.xen.org/xsa/)
