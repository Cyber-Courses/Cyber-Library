---
title: "Guest-to-host escape: breaking out of a Nutanix AHV virtual machine"
order: 2
description: "AHV runs VMs under KVM with QEMU-derived device emulation, so a guest-to-host escape targets the same QEMU device models, virtio, NICs, storage, USB, and lands in the per-VM QEMU process on the AHV host. From there the attacker reaches the node and, through the Controller VM relationship, the storage fabric and management plane."
keywords:
  - nutanix ahv escape
  - qemu
  - kvm
  - device model
  - controller vm
---

# Guest-to-host escape

Because AHV uses KVM with QEMU-derived device emulation, escaping an AHV guest is the QEMU problem: corrupt the per-VM QEMU process on the AHV node through a device model. The reachable surface and mechanisms are the QEMU ones, virtio virtqueues, the emulated NICs, the storage controllers, USB, so the per-device detail lives on the QEMU pages. What differs is the landing environment and the onward pivot: code execution lands on an AHV node whose storage and management flow through a Controller VM, so the escape is a step toward the node, the CVM, and the cluster.

```bash
# from the guest: the QEMU-emulated device inventory (escape surface)
lspci -nn; lsusb; dmesg | grep -i virtio
```

The device-model mechanisms are shared with QEMU:

- virtio virtqueue and indirect-descriptor handling, see [virtio devices](../qemu/guest-to-host-escape/virtio-devices.md).
- emulated NIC descriptor and offload handling, see [Network adapters](../qemu/guest-to-host-escape/network-adapters.md).
- storage controller command/PRD handling, see [Block and SCSI](../qemu/guest-to-host-escape/block-and-scsi.md).
- USB controller ring handling, see [USB controllers](../qemu/guest-to-host-escape/usb-controllers.md).

## What differs on AHV

```bash
# after landing in the QEMU process on the AHV node:
#  - the node's storage I/O is served by the local Controller VM; reaching the CVM
#    (over its internal network, typically 192.168.5.0/24) is the pivot to the fabric
#  - node and CVM credentials/keys enable moving to Prism and other nodes
ip route; ip -4 addr   # the internal CVM network is reachable from the host
```

## Exploitation notes

- The escape primitive and per-device mechanics are QEMU's; fingerprint the AHV (and thus QEMU) version and match to the device advisory, exactly as for [QEMU known escape exploits](../qemu/known-escape-exploits.md).
- The AHV-specific value is the onward path: from the node, the local Controller VM mediates all storage and runs management services, so a node foothold is a step toward the whole cluster via the CVM internal network.
- Nutanix adds its own confinement and services around QEMU; assess the node's sandboxing as with any KVM host, see [Host access and shell](host-access-and-shell.md).

## References

- [Nutanix AHV architecture](https://www.nutanix.com/products/ahv)
- [QEMU security advisories](https://www.qemu.org/docs/master/system/security.html)
- [Nutanix security advisories](https://www.nutanix.com/support-services/security-advisories)
