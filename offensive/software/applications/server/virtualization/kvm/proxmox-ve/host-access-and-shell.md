---
title: "Host access and shell: reaching a Proxmox node"
description: "Reaching a Proxmox VE node: root on the Debian host and the web interface on 8006, which together control every VM and container on the node and, through the cluster, the others. The pmxcfs filesystem at /etc/pve exposes the whole cluster configuration."
keywords:
  - Proxmox host
  - 8006
  - pmxcfs
  - /etc/pve
  - host access
---

# Host access and shell

A Proxmox node is a Debian host with a web UI on `8006` and a shell. Root on the host owns every VM and container on it, and the clustered configuration filesystem `pmxcfs` (mounted at `/etc/pve`) exposes the whole cluster's definitions, users, and tokens. Reaching the node is ordinary Linux compromise or a management-plane pivot.

```bash
qm list                                   # KVM guests on this node
pct list                                  # LXC containers on this node
qm terminal <vmid>                         # guest serial console
cat /etc/pve/user.cfg                      # users and roles (cluster-wide)
ls /etc/pve/qemu-server/                   # VM configs, disk references
```

## Exploitation notes

- `/etc/pve` is a cluster-wide view: user and token definitions, VM configs, and storage, readable with root on any node.
- `qm` and `pct` give console and full control of guests, and the storage is directly readable for [Disk and backup theft](disk-and-backup-theft.md).
- LXC containers are local to the node, so a container escape (standard Linux container-escape primitives) lands directly on the node.

## References

- [Proxmox VE: pmxcfs](https://pve.proxmox.com/wiki/Proxmox_Cluster_File_System_(pmxcfs))
- [Proxmox VE administration guide](https://pve.proxmox.com/pve-docs/)
