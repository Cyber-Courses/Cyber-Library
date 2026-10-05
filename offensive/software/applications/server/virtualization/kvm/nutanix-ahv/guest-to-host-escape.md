---
title: "Guest to host escape: breaking out of a Nutanix AHV guest"
description: "Escaping a Nutanix AHV guest to the host: AHV runs guests on QEMU/KVM, so a breakout is a QEMU device-model escape that lands on the AHV host, from which the Controller VM and the storage fabric, and through them the cluster, become reachable."
keywords:
  - Nutanix escape
  - AHV
  - QEMU
  - CVM
  - guest to host
---

# Guest to host escape

AHV runs its guests on QEMU and KVM, so escaping an AHV VM is a QEMU device-model escape, identical in surface and technique to any KVM host. Code execution lands in the QEMU process on the AHV host. From the host, the local Controller VM and the storage fabric it serves are reachable, which is the path from one VM to cluster-wide impact.

```text
Nutanix AHV guest escape surface:
- The QEMU device models (virtio, NICs, USB, SCSI) -> see KVM/QEMU
- From the AHV host: the local CVM and the storage fabric
```

## Exploitation notes

- The technique and surface are the [KVM and QEMU guest to host escape](../qemu/guest-to-host-escape.md), bounded by the AHV host's QEMU confinement.
- The escape's value is the pivot: AHV host to CVM to the distributed storage, which holds every VM's disks for [Disk and snapshot theft](disk-and-snapshot-theft.md).
- Nutanix-specific and QEMU named issues are under [Known escape exploits](known-escape-exploits.md).

## References

- [Nutanix AHV architecture](https://www.nutanix.dev/)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
