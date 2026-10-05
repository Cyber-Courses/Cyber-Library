---
title: "Hyper-V: attacking the Windows hypervisor and its management"
description: "Attacking Microsoft Hyper-V: reaching the parent-partition host, escaping a guest to the host through the virtualization stack (vmbus, the virtual switch, the VM worker process), abusing the WMI and PowerShell management plane, and stealing checkpoints and VHD disks."
keywords:
  - Hyper-V
  - VM escape
  - vmbus
  - WMI
  - VHD
---

# Hyper-V

Hyper-V is a type-1 hypervisor that runs the Windows host as a privileged parent partition managing child-partition guests. Offensive interest is in reaching that parent partition, breaking out of a child partition through the synthetic device stack (VMBus, the virtual switch, the `vmwp.exe` worker process), controlling guests through the WMI and PowerShell management plane, and lifting checkpoints and VHD files.

## Subtopics

- **[Host access and shell](host-access-and-shell.md)**: reaching the parent partition.
- **[Guest to host escape](guest-to-host-escape/index.md)**: breaking out of a child partition.
- **[Management plane and WMI abuse](management-plane-and-wmi-abuse.md)**: controlling guests through management interfaces.
- **[Checkpoint and VHD theft](checkpoint-and-vhd-theft.md)**: lifting guest disks and checkpoints.
- **[Known escape exploits](known-escape-exploits.md)**: named Hyper-V breakouts.

## References

- [Microsoft: Hyper-V architecture](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [Microsoft Hyper-V bug bounty](https://www.microsoft.com/en-us/msrc/bounty-hyper-v)
