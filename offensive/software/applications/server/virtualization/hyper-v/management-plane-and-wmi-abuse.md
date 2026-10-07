---
title: "Management plane and WMI abuse: controlling Hyper-V through its APIs"
order: 2
description: "Hyper-V is managed through WMI (the root/virtualization namespace), PowerShell cmdlets, and remote services. An attacker with rights to that management surface controls every VM, creates or modifies VMs to mount host-reachable disks or run payloads, uses PowerShell Direct into guests, and plants WMI event subscriptions as host persistence."
keywords:
  - hyper-v wmi
  - root/virtualization
  - powershell direct
  - wmi event subscription
  - management plane
---

# Management plane and WMI abuse

Hyper-V's control surface is programmatic: the WMI `root/virtualization/v2` namespace, the Hyper-V PowerShell module, and the remote management services (WinRM, RPC/DCOM) that front them. An attacker who can reach that surface with sufficient rights controls the virtualization estate without a guest escape: enumerate and manipulate VMs, create or reconfigure a VM to mount a disk or run a payload, drive guests with PowerShell Direct, and use WMI's eventing as durable host persistence.

## Drive Hyper-V through WMI and PowerShell

```powershell
# enumerate and control VMs via the Hyper-V module (wraps the WMI namespace)
Get-VM; Get-VMHardDiskDrive -VMName *
# or directly through WMI
Get-CimInstance -Namespace root\virtualization\v2 -ClassName Msvm_ComputerSystem
# create/modify a VM to attach a target VHD or boot attacker media
New-VM -Name x -VHDPath '<target>.vhdx'          # attach another VM's disk
# run inside a guest from the host, no guest network needed
Invoke-Command -VMName <guest> -Credential <c> -ScriptBlock { ... }
```

## WMI as persistence

```powershell
# a permanent WMI event subscription runs a payload on a trigger, as SYSTEM, and
# survives reboots - a classic host persistence mechanism on the Hyper-V host
# (__EventFilter + CommandLineEventConsumer + FilterToConsumerBinding)
```

## Exploitation notes

- The management plane is an alternative to a guest escape: with host admin or delegated Hyper-V rights, you control every VM and can attach another VM's disk to a VM you control to read it, or boot a VM from attacker media.
- PowerShell Direct is the quiet host-to-guest pivot: it runs in the guest over the VMBus-backed channel with no guest network exposure, needing only guest credentials and host access.
- WMI permanent event subscriptions are durable SYSTEM-level persistence on the host, and blend in as administrative tooling.
- Delegated management (Hyper-V Administrators group, constrained delegation) can grant VM control without full host admin; enumerate who holds it as a targeting aid.

## References

- [Microsoft: Hyper-V WMI provider (root/virtualization)](https://learn.microsoft.com/en-us/windows/win32/hyperv_v2/windows-virtualization-portal)
- [Microsoft: Hyper-V PowerShell module](https://learn.microsoft.com/en-us/powershell/module/hyper-v/)
- [MITRE ATT&CK: WMI event subscription](https://attack.mitre.org/techniques/T1546/003/)
