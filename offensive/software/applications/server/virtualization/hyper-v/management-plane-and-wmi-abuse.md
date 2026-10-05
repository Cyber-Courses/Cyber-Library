---
title: "Management plane and WMI abuse: controlling Hyper-V guests"
description: "Abusing the Hyper-V management plane to control guests without an escape: the WMI virtualization namespace, Failover Cluster and System Center Virtual Machine Manager, and Windows Admin Center, which let an attacker with host or management rights manipulate, mount, and interact with any VM."
keywords:
  - Hyper-V WMI
  - SCVMM
  - Failover Cluster
  - virtualization namespace
  - management plane
---

# Management plane and WMI abuse

Hyper-V is managed through WMI (the `root\virtualization\v2` namespace), Failover Clustering, System Center Virtual Machine Manager (SCVMM), and Windows Admin Center. An attacker with rights on the host or these management services controls VMs directly: starting, stopping, modifying, injecting into, and mounting their disks, without touching a guest-to-host exploit.

```powershell
# WMI virtualization namespace: enumerate and act on VMs
Get-CimInstance -Namespace root\virtualization\v2 -ClassName Msvm_ComputerSystem |
  Select ElementName, EnabledState
# Modify a VM (e.g., attach a disk, change boot) via the management service
```

## Exploitation notes

- SCVMM is a high-value target: it manages many hosts, and its service account and database reach the whole virtualization estate.
- WMI access to the virtualization namespace is control of VMs; it is also a quiet way to mount a guest disk offline for [Checkpoint and VHD theft](checkpoint-and-vhd-theft.md).
- Failover Cluster compromise moves control across every clustered Hyper-V host at once.

## References

- [Microsoft: Hyper-V WMI provider](https://learn.microsoft.com/en-us/windows/win32/hyperv_v2/windows-virtualization-portal)
- [Microsoft: System Center VMM](https://learn.microsoft.com/en-us/system-center/vmm/)
