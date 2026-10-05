---
title: "Disk and snapshot theft: reading KVM qcow2 disks offline"
description: "Stealing KVM guest data by copying qcow2 and raw disk images and their internal snapshots from the host or shared storage, then mounting them offline to extract credentials and files, or reading a running guest's memory from a saved-state file."
keywords:
  - qcow2
  - raw image
  - offline disk
  - guestfish
  - snapshot
---

# Disk and snapshot theft

KVM guest disks are `qcow2` or raw images on the host, usually under `/var/lib/libvirt/images`, often on shared or network storage. With host or storage access, copying an image and mounting it offline exposes the guest filesystem and its secrets, and qcow2 internal snapshots and libvirt save files capture point-in-time state including memory.

```bash
# Inspect and mount a guest image offline
qemu-img info disk.qcow2
guestfish --ro -a disk.qcow2 -i          # browse the guest filesystem
# or via nbd
qemu-nbd -r -c /dev/nbd0 disk.qcow2 && mount -o ro /dev/nbd0p1 /mnt
```

## Exploitation notes

- Offline access sidesteps the guest OS: pull Linux shadow files or Windows `SAM`/`NTDS.dit`, then crack or reuse the credentials.
- `libguestfs` (`guestfish`, `virt-cat`) reads images without mounting, which is quiet and needs no loop or nbd device.
- A `virsh save` state file and qcow2 snapshots can contain live memory, so they may hold secrets from a running guest.

## References

- [libguestfs](https://libguestfs.org/)
- [QEMU: qcow2 and qemu-img](https://www.qemu.org/docs/master/tools/qemu-img.html)
