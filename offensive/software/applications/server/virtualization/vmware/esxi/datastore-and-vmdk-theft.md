---
title: "Datastore and VMDK theft: stealing ESXi guest disks offline"
description: "Stealing VMware guest data by copying VMDK virtual disks from the ESXi datastore, with host or storage access, then mounting them offline to extract the SAM, NTDS.dit, and files without booting the guest, or by registering and booting a copied VM elsewhere."
keywords:
  - VMDK
  - datastore
  - VMFS
  - offline disk
  - NTDS.dit
---

# Datastore and VMDK theft

A VMware guest's disk is a `VMDK` pair (a small descriptor and a large `-flat` or sparse extent) on a VMFS or NFS datastore. With ESXi shell, API, or storage access, copying the VMDK and mounting it offline exposes the guest filesystem and its secrets, or the whole VM can be registered and booted on attacker infrastructure.

```bash
# On the ESXi host: locate and export a guest disk
ls /vmfs/volumes/<datastore>/<vm>/
# Download via the datastore browser (vSphere API) or scp, then mount offline:
qemu-nbd -r -c /dev/nbd0 vm-flat.vmdk && mount -o ro /dev/nbd0p1 /mnt
# Extract SAM/SYSTEM or NTDS.dit from the mounted volume
```

## Exploitation notes

- Offline access sidesteps the guest OS: pull `SAM`/`SYSTEM`, or `NTDS.dit` from a domain controller VM, then crack or pass the hashes.
- Snapshots (`-delta.vmdk`) capture point-in-time state and memory (`.vmsn`/`.vmem`), which can hold live secrets.
- Thin extents may need consolidation; `vmkfstools -i` on the host clones a VMDK into a portable image.

## References

- [VMware: vmkfstools](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-storage/GUID-A5D85C33-A510-4A3E-8FC7-93E6BA0A048F.html)
- [VMware: virtual disk format](https://docs.vmware.com/en/VMware-vSphere/index.html)
