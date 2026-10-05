---
title: "Proxmox VE: attacking the Debian, KVM, and LXC platform"
description: "Attacking Proxmox VE: reaching the node host and its web and REST API, escaping a guest to the node through the underlying QEMU and KVM device models, abusing the cluster API and authentication, and stealing virtual disks and vzdump backups. Proxmox has become a leading ESXi alternative."
keywords:
  - Proxmox VE
  - pvesh
  - KVM
  - cluster
  - VM escape
---

# Proxmox VE

Proxmox VE is Debian plus KVM and LXC with a clustered web and REST management layer. It has become a leading destination for teams leaving ESXi, so it is an increasingly common target. Offensively, it is a Debian host (root there owns everything), a web and API control plane with its own ticket and token authentication, QEMU/KVM-backed guests with the usual escape surface, and virtual disks and backups at rest.

## Subtopics

- **[Host access and shell](host-access-and-shell.md)**: reaching the node.
- **[Guest to host escape](guest-to-host-escape.md)**: breaking out through QEMU/KVM.
- **[Management plane](management-plane.md)**: the cluster API and authentication.
- **[Disk and backup theft](disk-and-backup-theft.md)**: disks, snapshots, and vzdump backups.
- **[Known escape exploits](known-escape-exploits.md)**: named Proxmox and QEMU breakouts.

## References

- [Proxmox VE documentation](https://pve.proxmox.com/pve-docs/)
- [Proxmox VE API](https://pve.proxmox.com/pve-docs/api-viewer/)
