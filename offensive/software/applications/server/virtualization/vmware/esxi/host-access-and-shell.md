---
title: "Host access and shell: reaching the ESXi hypervisor"
description: "Reaching an ESXi host: its SSH and ESXi Shell, the host client and vSphere API on 443, and the vpxuser and root accounts, which grant full control of every VM on the host including console access and offline disk theft."
keywords:
  - ESXi shell
  - ESXi SSH
  - vpxuser
  - vSphere API
  - host access
---

# Host access and shell

ESXi is reached through SSH and the ESXi Shell, the host client and vSphere API on `443`, and the DCUI. Control of the host is control of every VM on it. Credentials come from brute-forcing or spraying `root`, from a compromised vCenter (which stores the per-host `vpxuser` password it uses to manage each host), or from host config backups.

```bash
ssh root@<esxi>                             # ESXi Shell
vim-cmd vmsvc/getallvms                      # list VMs
vim-cmd vmsvc/power.off <vmid>               # control a VM
esxcli system account list                   # local accounts
# vSphere API over 443 (pyVmomi, govc) with host or vpxuser creds
govc ls -u 'root:pass@<esxi>' -k /
```

## Exploitation notes

- A compromised vCenter yields `vpxuser` credentials for every managed host, so vCenter-to-ESXi is a one-step pivot; see [vCenter](../vcenter/index.md).
- From the shell, `vim-cmd` and the datastore give console access and direct `VMDK` access for [Datastore and VMDK theft](datastore-and-vmdk-theft.md).
- ESXi mass-encryption intrusions typically start exactly here: SSH or API access, then encrypt datastores.

## References

- [VMware: using the ESXi Shell](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-security/GUID-70557A95-2B1A-4A66-ADC0-6F4A4A4B6B6E.html)
- [govc CLI](https://github.com/vmware/govmomi/tree/main/govc)
