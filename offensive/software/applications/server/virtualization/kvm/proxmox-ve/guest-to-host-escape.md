---
title: "Guest-to-host escape: breaking out of a Proxmox virtual machine"
description: "Proxmox runs VMs under KVM with QEMU device emulation, so a VM guest-to-host escape targets the same QEMU device models and lands in the per-VM QEMU process on the Proxmox node. Proxmox also runs LXC containers, whose escape is the runtime-agnostic container breakout rather than a device-model bug."
keywords:
  - proxmox escape
  - qemu
  - kvm
  - lxc
  - device model
---

# Guest-to-host escape

Proxmox VMs run under KVM with QEMU device emulation, so escaping a Proxmox VM is the QEMU problem: corrupt the per-VM QEMU process on the node through a device model. The reachable devices and mechanisms are QEMU's, documented on the QEMU pages, and code execution lands on the Proxmox node. Separately, Proxmox also runs LXC containers, and escaping one of those is not a device-model bug at all but the runtime-agnostic container escape (over-privileged config, sensitive mounts, shared namespaces, or a kernel exploit).

```bash
# VM guest: the QEMU-emulated device inventory (escape surface)
lspci -nn; lsusb; dmesg | grep -i virtio
# LXC container: check the container-escape surface instead
grep CapEff /proc/self/status; cat /proc/self/uid_map; mount | grep -v overlay | head
```

## VM escape: the QEMU surface

The device-model mechanisms are shared with QEMU:

- virtio virtqueue handling, see [virtio devices](../qemu/guest-to-host-escape/virtio-devices.md).
- emulated NIC handling, see [Network adapters](../qemu/guest-to-host-escape/network-adapters.md).
- storage controller handling, see [Block and SCSI](../qemu/guest-to-host-escape/block-and-scsi.md).
- USB controller handling, see [USB controllers](../qemu/guest-to-host-escape/usb-controllers.md).

## LXC container escape

```bash
# Proxmox LXC containers escape via the container primitives, not device models:
# privileged containers, bind mounts of host paths, shared namespaces, kernel bugs
# unprivileged LXC uses a user namespace, which constrains what an escape yields
cat /proc/self/uid_map     # "0 100000 65536" => unprivileged container
```

The container route is the runtime-agnostic [Container escape](../../../containers/container-escape/index.md); Proxmox privileged containers are the most exposed, and unprivileged ones (user-namespaced) constrain the result.

## Exploitation notes

- For VMs, treat Proxmox as QEMU: fingerprint the QEMU version and match a device advisory, landing in the per-VM QEMU process on the node.
- For LXC, the escape is container breakout; a Proxmox privileged container that also mounts host paths or shares namespaces is the easy case, and the mechanisms are on the container-escape pages.
- Either way the landing point is the Proxmox node; continue with [Host access and shell](host-access-and-shell.md) to reach the management and cluster.

## References

- [Proxmox VE: VMs and containers](https://pve.proxmox.com/pve-docs/)
- [QEMU security advisories](https://www.qemu.org/docs/master/system/security.html)
