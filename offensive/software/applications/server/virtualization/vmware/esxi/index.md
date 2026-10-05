---
title: "ESXi: attacking the VMware hypervisor host"
description: "Attacking a VMware ESXi host: reaching its shell and management API, escaping a guest to the host through VMX device emulation, and stealing guest VMDK disks from the datastore. ESXi hosts have become a prime ransomware and intrusion target."
keywords:
  - ESXi
  - VMX
  - VMFS
  - VMDK
  - hypervisor host
---

# ESXi

ESXi is a thin, hardened hypervisor managed through a host client, SSH, and the vSphere API. Each VM is served by a `vmx` user-space process that emulates its devices, which is the guest-to-host escape surface, while the datastore holds every VM's `VMDK` disk. ESXi hosts are now a favored target for mass VM encryption, so access patterns and offline disk theft matter as much as escapes.

## Subtopics

- **[Host access and shell](host-access-and-shell.md)**: reaching the ESXi shell and API.
- **[Guest to host escape](guest-to-host-escape/index.md)**: breaking out through VMX device emulation.
- **[Datastore and VMDK theft](datastore-and-vmdk-theft.md)**: stealing guest disks offline.
- **[Known escape exploits](known-escape-exploits.md)**: named ESXi breakouts.

## References

- [VMware: ESXi security](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-security/GUID-61ABA2A2-0B6B-4F9A-8F4B-1F4B2B5F5D3F.html)
- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
