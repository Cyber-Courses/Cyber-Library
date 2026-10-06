---
title: "ESXi: attacking the bare-metal VMware hypervisor"
order: 2
description: "ESXi runs virtual machines directly on hardware under the vmkernel, with each VM backed by a vmx process. The offensive surface is the guest-to-host escape through emulated devices, theft of virtual disks and data from datastores, obtaining a host shell through the management services, and the recurring named escape exploits from device-model flaws."
keywords:
  - esxi
  - vmkernel
  - vmx
  - datastore
  - hypervisor
---

# ESXi

ESXi is VMware's type-1 hypervisor: the `vmkernel` runs on the hardware, and each virtual machine is backed by a user-space `vmx` process that emulates its devices. Offensively it is attacked from two directions. From inside a guest, the target is the `vmx` device emulation, a memory-safety escape to the host. From the host or management network, the targets are the datastores that hold every VM's disks and the management services that give a host shell. ESXi is a high-value target because one host runs many VMs, so host compromise exposes all of them.

```bash
# from a guest: emulated device inventory (escape surface)
lspci -nn; lsusb
# from the network: ESXi management surface
nmap -p 22,443,427,902 <esxi-host>        # SSH, host UI/API, CIM/SLP, vmx authd
curl -sk https://<esxi-host>/            # host client / API reachable
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape/index.md)**: breaking out of a VM through emulated devices.
- **[Datastore and VMDK theft](datastore-and-vmdk-theft.md)**: stealing virtual disks and their data.
- **[Host access and shell](host-access-and-shell.md)**: obtaining execution on the ESXi host.
- **[Known escape exploits](known-escape-exploits.md)**: the recurring device-model escapes.

## References

- [VMware ESXi documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
