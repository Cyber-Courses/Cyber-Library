---
title: "Proxmox VE: attacking the KVM and LXC virtualization platform"
order: 3
description: "Proxmox VE is a Debian-based platform running KVM/QEMU virtual machines and LXC containers, managed through a web UI and REST API on port 8006 and clustered with corosync. The guest-to-host escape inherits QEMU's device models; the distinctive surface is the management API and web UI, the cluster, and the storage backends holding VM disks."
keywords:
  - proxmox ve
  - qemu
  - lxc
  - pvedaemon
  - corosync
---

# Proxmox VE

Proxmox VE is a Debian-based virtualization platform that runs both KVM/QEMU virtual machines and LXC containers, administered through a web UI and REST API (the `pvedaemon`/`pveproxy` services on port 8006) and clustered with corosync. Its attack surface has three parts. The guest-to-host escape from a Proxmox VM is the QEMU device-model problem, inherited wholesale. The management plane, the API, web UI, and the `pvesh`/`qm`/`pct` tooling, controls every VM and container. And the storage backends (local directories, LVM, ZFS, Ceph) hold the VM disks and backups. LXC containers add the container-escape surface alongside the VM one.

```bash
# on a Proxmox node
qm list; pct list                         # KVM VMs and LXC containers
ss -tlnp | grep 8006                       # the management web/API
pvecm status 2>/dev/null                   # cluster membership
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape.md)**: breaking out of a Proxmox VM via QEMU.
- **[Host access and shell](host-access-and-shell.md)**: execution on the Proxmox node.
- **[Management plane](management-plane.md)**: the API, web UI, and cluster.
- **[Disk and backup theft](disk-and-backup-theft.md)**: taking VM disks and vzdump backups.
- **[Known escape exploits](known-escape-exploits.md)**: the inherited QEMU and platform bugs.

## References

- [Proxmox VE documentation](https://pve.proxmox.com/pve-docs/)
- [Proxmox VE API viewer](https://pve.proxmox.com/pve-docs/api-viewer/)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
