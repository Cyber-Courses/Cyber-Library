---
title: "Disk and snapshot theft: reading Nutanix guest disks"
description: "Stealing Nutanix guest data by reading virtual disks and snapshots from the distributed storage fabric, through the Controller VM, the container shares the fabric exposes over NFS and SMB, and Prism snapshot operations, then mounting them offline to extract secrets."
keywords:
  - Nutanix storage
  - vdisk
  - storage container
  - snapshot
  - offline disk
---

# Disk and snapshot theft

Nutanix guest disks (vdisks) live in the distributed storage fabric, presented through the Controller VMs and exposed as storage containers over NFS and SMB. With CVM access, container-share access, or Prism, guest disks and snapshots can be read or cloned, then mounted offline to extract credentials and files without booting the guest.

```bash
# From the CVM: list vdisks and storage, or reach the container shares
acli vm.disk_get <vm>
# Storage containers are often exposed over NFS/SMB to the hosts
showmount -e <cvm>            # NFS exports (container shares)
```

## Exploitation notes

- The storage fabric holds every VM's disks, so CVM or container-share access generalizes to many guests at once.
- Prism snapshot and clone operations create readable copies of guest disks through the management plane.
- Offline access sidesteps the guest OS: pull Linux shadow files or Windows `SAM`/`NTDS.dit`.

## References

- [Nutanix: storage containers](https://portal.nutanix.com/)
- [Nutanix AHV administration](https://portal.nutanix.com/)
