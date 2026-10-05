---
title: "Host access and shell: reaching the Hyper-V parent partition"
description: "Reaching the Hyper-V host, the Windows parent partition, which has full control of every guest: through normal Windows access to the host, the Hyper-V Manager and PowerShell, or a management service, from where guest disks and consoles are available."
keywords:
  - Hyper-V host
  - parent partition
  - Hyper-V Manager
  - PowerShell Direct
  - host access
---

# Host access and shell

The Hyper-V host is a Windows machine (the parent partition), and control of it is control of every guest. Reaching it is ordinary Windows compromise: credentials for the host, the `Hyper-V Administrators` group, or a management service. Once on the host, guests are fully exposed through PowerShell and the console.

```powershell
Get-VM                                   # every guest on this host
Get-VMHost | fl                          # host and hypervisor settings
# Interact with a guest OS directly from the host (no network needed)
Enter-PSSession -VMName <guest> -Credential <cred>   # PowerShell Direct
Get-VMHardDiskDrive -VMName <guest>      # locate its VHDs
```

## Exploitation notes

- Membership in `Hyper-V Administrators` is host-level control of all guests without local admin on the host itself.
- PowerShell Direct reaches a guest OS from the host over VMBus with no guest network, useful for guests isolated on the network.
- The host holds every guest's VHD and checkpoints, so host access leads directly to [Checkpoint and VHD theft](checkpoint-and-vhd-theft.md).

## References

- [Microsoft: manage Hyper-V with PowerShell](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/manage-hyper-v-with-powershell)
- [Microsoft: PowerShell Direct](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/powershell-direct)
