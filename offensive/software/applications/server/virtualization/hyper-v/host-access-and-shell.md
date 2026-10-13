---
title: "Host access and shell: execution on the Hyper-V host"
order: 4
description: "A Hyper-V host is a Windows Server (or client) machine, so host access is Windows access: administrative credentials, the management interfaces, or a guest escape landing in the root partition. With it, an attacker controls every VM through Hyper-V management, reads VM disks, injects into guests, and persists on the host like any Windows system."
keywords:
  - hyper-v host
  - root partition
  - windows server
  - hyper-v manager
  - persistence
---

# Host access and shell

A Hyper-V host is a Windows machine running the role, so reaching the host is reaching Windows: through administrative credentials and the usual remote interfaces (RDP, WinRM, SMB, WMI), through the Hyper-V management stack, or through a guest escape that lands in the root partition. The root partition is a privileged Windows partition that administers every guest, so host code execution means full control of all VMs, their disks, and the virtualization configuration, with persistence options identical to any Windows server.

## Reach and use the host

```powershell
# remote execution with host admin credentials
Enter-PSSession -ComputerName <host> -Credential <admin>
winrs -r:<host> cmd                              # or WinRM / PsExec / WMI
# once on the host, Hyper-V management controls every VM
Get-VM; Get-VMHardDiskDrive -VMName *
# run commands inside a guest from the host (with guest creds or via PowerShell Direct)
Invoke-Command -VMName <guest> -ScriptBlock { whoami } -Credential <guestcred>
```

PowerShell Direct (`-VMName`) runs commands in a guest from the host without network access to the guest, using the host's privileged position, which is a clean host-to-guest pivot.

## Persistence

```powershell
# the host is Windows: standard persistence applies
#  - a service, scheduled task, or Run key
#  - a WMI event subscription (see management-plane-and-wmi-abuse)
# plus virtualization-specific leverage: modify a VM's config or inject via its disk
```

## Exploitation notes

- Host access subsumes everything else: from the root partition you read and modify VM disks ([Checkpoint and VHD theft](checkpoint-and-vhd-theft.md)) and run inside guests via PowerShell Direct without needing guest network reach.
- A guest-to-host escape lands code in the root partition or worker process; from there, local privilege escalation to full host admin follows the normal Windows playbook if not already SYSTEM.
- Persistence is standard Windows persistence; the virtualization-specific addition is tampering with VM configurations and disks so a target VM runs attacker content on next boot.
- The management plane (WMI/PowerShell) is both a remote entry and a persistence surface; see [Management plane and WMI abuse](management-plane-and-wmi-abuse.md).

## References

- [Microsoft: Hyper-V management](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/)
- [Microsoft: PowerShell Direct](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/powershell-direct)
