---
title: "Disk and snapshot theft: reading Xen guest disks offline"
description: "Stealing Xen guest data by reading virtual disks and snapshots from dom0 or the storage repository, whether LVM volumes, VHD files, or raw images, then mounting them offline to extract credentials and files without booting the guest."
keywords:
  - Xen disk
  - storage repository
  - VHD
  - LVM
  - offline disk
---

# Disk and snapshot theft

Xen guest disks live on a storage repository managed by dom0: LVM logical volumes, VHD files, or raw images depending on the storage type. With dom0 or storage access, exporting a virtual disk and mounting it offline exposes the guest filesystem, and snapshots capture point-in-time state.

```bash
# XCP-ng/XenServer: export a virtual disk image
xe vdi-list; xe vdi-export uuid=<vdi-uuid> filename=disk.raw format=raw
# Mount offline to extract secrets
qemu-nbd -r -c /dev/nbd0 disk.raw && mount -o ro /dev/nbd0p1 /mnt
```

## Exploitation notes

- Offline access sidesteps the guest OS: pull Linux shadow files or Windows `SAM`/`NTDS.dit`, then crack or reuse them.
- On LVM-backed storage, the guest volume can be read directly from dom0 with standard tools once its name is known.
- Snapshots and suspended-state files can contain memory, so they may hold secrets from a running guest.

## References

- [XCP-ng: storage](https://docs.xcp-ng.org/storage/)
- [Xen Project documentation](https://xenproject.org/help/documentation/)
