---
title: "Host access and shell: execution on the Proxmox node"
order: 1
description: "A Proxmox node is a Debian host running the VMs, containers, and management daemons as root. Access comes from a guest or container escape, SSH, or the management API and web shell. With it, an attacker controls every VM and container through qm/pct, reads all disks and backups, and in a cluster reaches the other nodes through the shared corosync-backed configuration."
keywords:
  - proxmox node
  - pvedaemon
  - qm pct
  - corosync
  - cluster
---

# Host access and shell

A Proxmox node is a Debian server where the management daemons (`pvedaemon`, `pveproxy`, `pve-cluster`) and the VM/container processes run as root. Reaching a shell comes from a guest or container escape landing on the node, from SSH with node credentials, or from the management interface, which includes a built-in web shell (noVNC/xterm.js) and command execution via the API. Node root is control of every VM and container on it through `qm` and `pct`, access to all disks and backups, and, in a cluster, reach to the other nodes because the cluster filesystem and corosync tie them together.

## Reach and use the node

```bash
# SSH with node credentials, or the web-UI shell
ssh root@<proxmox-node>
qm list; pct list                         # VMs and containers
# run inside a VM via the guest agent, or in a container directly
qm guest exec <vmid> -- id 2>/dev/null
pct exec <ctid> -- id
```

## Cluster reach

```bash
# the cluster filesystem (pmxcfs) is shared across nodes under /etc/pve
pvecm status; pvecm nodes
cat /etc/pve/corosync.conf                 # cluster members
ls /etc/pve/nodes/                         # per-node config, shared cluster-wide
# node root on one member, plus the shared /etc/pve and SSH trust, reaches peers
ssh root@<other-node>                      # cluster nodes commonly trust each other
```

`/etc/pve` is the corosync-backed cluster filesystem (pmxcfs) replicated to every node, and cluster members typically share root SSH trust, so compromising one node commonly extends to the whole cluster.

## Exploitation notes

- Node root subsumes everything: `qm`/`pct` control all guests, the guest agent runs commands inside VMs, and `pct exec` enters containers directly.
- The shared cluster filesystem under `/etc/pve` and the usual inter-node SSH trust make a single-node compromise a cluster compromise; enumerate peers via `pvecm nodes` and the shared config.
- The web UI's shell and the API's command execution are legitimate node-access paths with management credentials, an alternative to a guest escape.
- Persistence is standard Debian (systemd, cron, SSH keys); the cluster filesystem offers a replicated location visible to all nodes.

## References

- [Proxmox VE: cluster manager (pmxcfs/corosync)](https://pve.proxmox.com/pve-docs/chapter-pvecm.html)
- [Proxmox VE administration](https://pve.proxmox.com/pve-docs/)
