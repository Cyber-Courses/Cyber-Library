---
title: "Guest to host escape: breaking out of a Proxmox guest"
description: "Escaping a Proxmox guest to the node: VMs run on QEMU/KVM, so they inherit the QEMU device-model escape surface, while LXC containers use the standard Linux container-escape primitives, either of which lands on the Debian node and then pivots across the cluster."
keywords:
  - Proxmox escape
  - QEMU
  - LXC
  - node breakout
  - guest to host
---

# Guest to host escape

Proxmox runs two kinds of guest. VMs are QEMU/KVM, so escaping one is a QEMU device-model escape that lands in the QEMU process on the node. Containers are LXC, so escaping one uses the standard Linux container-escape primitives (privileged config, dangerous mounts, host namespaces). Either outcome is code execution on the Debian node, from which the cluster is reachable.

```text
Proxmox guest escape surfaces:
- VMs: the QEMU device models (virtio, NICs, USB, SCSI) -> see KVM/QEMU
- LXC: privileged containers, bind mounts, shared namespaces -> see Container escape
```

## Exploitation notes

- For VMs, the technique and surface are identical to [KVM and QEMU guest to host escape](../qemu/guest-to-host-escape.md), bounded by the node's QEMU confinement.
- For LXC, especially privileged containers, the breakout is the generic [Container escape](../../../containers/container-escape/index.md), and it lands directly on the node.
- Once on the node, `/etc/pve` and the cluster communication give lateral movement to other nodes; see [Management plane](management-plane.md).

## References

- [Proxmox VE: Linux containers](https://pve.proxmox.com/wiki/Linux_Container)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
