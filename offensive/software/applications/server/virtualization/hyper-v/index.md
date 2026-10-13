---
title: "Hyper-V: attacking the Microsoft hypervisor"
order: 1
description: "Hyper-V is a type-1 hypervisor where guests communicate with the root partition over VMBus. The guest-to-host escape surface is the synthetic devices and the VMBus channels serviced by the VSPs in the root partition and the worker process, plus the management plane through WMI and PowerShell, and theft of checkpoints and VHD virtual disks."
keywords:
  - hyper-v
  - vmbus
  - root partition
  - vmwp
  - synthetic devices
---

# Hyper-V

Hyper-V is Microsoft's type-1 hypervisor: a thin hypervisor runs beneath a privileged root partition, and guest partitions reach host services over VMBus, a ring-buffer channel transport. Each VM has a worker process (`vmwp.exe`) in the root partition, and the synthetic devices a guest uses are serviced by Virtualization Service Providers (VSPs) there. The guest-to-host escape surface is that VMBus and synthetic-device handling in the root partition; alongside it are the management plane (WMI and PowerShell cmdlets) and the on-disk artefacts, checkpoints and VHD/VHDX virtual disks, that hold guest data.

```powershell
# from a Windows guest: the synthetic devices and VMBus
Get-PnpDevice | Where-Object { $_.InstanceId -like 'VMBUS*' }
# from the host: VMs, their worker processes, and management
Get-VM; Get-Process vmwp
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape/index.md)**: VMBus and synthetic-device breakout to the root partition.
- **[Checkpoint and VHD theft](checkpoint-and-vhd-theft.md)**: stealing virtual disks and saved state.
- **[Host access and shell](host-access-and-shell.md)**: execution on the Hyper-V host.
- **[Management plane and WMI abuse](management-plane-and-wmi-abuse.md)**: the WMI and PowerShell control surface.
- **[Known escape exploits](known-escape-exploits.md)**: the recurring VMBus and VSP bugs.

## References

- [Microsoft: Hyper-V architecture](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [MSRC: Hyper-V bug bounty and research](https://www.microsoft.com/en-us/msrc)
- [Microsoft security update guide](https://msrc.microsoft.com/update-guide)
