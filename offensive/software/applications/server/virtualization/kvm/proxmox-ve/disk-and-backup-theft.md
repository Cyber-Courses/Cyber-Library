---
title: "Disk and backup theft: reading Proxmox disks and vzdump backups"
description: "Stealing Proxmox guest data by reading virtual disks and snapshots from node and cluster storage, and by looting vzdump backups, which are full guest images often kept on an accessible backup server, then mounting them offline to extract credentials and files."
keywords:
  - Proxmox disk
  - vzdump
  - Proxmox Backup Server
  - qcow2
  - offline disk
---

# Disk and backup theft

Proxmox stores VM disks as qcow2, raw, or LVM volumes on the node's configured storage, and it takes full-guest backups with `vzdump`, commonly to an NFS share or a Proxmox Backup Server. Both are offline paths to guest data: mount a disk image, or restore and read a backup, without booting or authenticating to the guest.

```bash
ls /var/lib/vz/images/<vmid>/              # local VM disks
ls /var/lib/vz/dump/                        # vzdump backup archives (vma/tar)
# Mount a disk image offline to extract secrets
guestfish --ro -a vm-disk.qcow2 -i
```

## Exploitation notes

- `vzdump` archives are complete guest images; a reachable backup share or Proxmox Backup Server holds every protected VM's data.
- Offline disk access sidesteps the guest OS: pull Linux shadow files or Windows `SAM`/`NTDS.dit`.
- Backup credentials and encryption keys in `/etc/pve` or the backup client config widen access to the whole backup store.

## References

- [Proxmox VE: vzdump backup](https://pve.proxmox.com/pve-docs/vzdump.1.html)
- [Proxmox Backup Server](https://pbs.proxmox.com/docs/)
