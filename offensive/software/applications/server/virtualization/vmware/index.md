---
title: "VMware: attacking ESXi, vCenter, and the desktop hypervisors"
order: 2
description: "VMware's virtualization stack spans the bare-metal ESXi hypervisor, the vCenter management plane that controls fleets of hosts, and the desktop products Workstation and Fusion. The offensive themes are consistent: guest-to-host escapes through shared device-emulation code, theft of virtual disks, and compromise of the management plane."
keywords:
  - vmware
  - esxi
  - vcenter
  - workstation
  - fusion
---

# VMware

VMware's products share a lineage and much of their code, so their attack surfaces rhyme. ESXi is the type-1 hypervisor running VMs on bare metal; vCenter is the management plane controlling many ESXi hosts; and Workstation and Fusion are the type-2 desktop hypervisors on Windows and macOS. The same device-emulation code (SVGA, USB, NICs, the GuestRPC and VMCI channels) appears across them, so a guest-to-host escape bug class in one often applies to the others. Alongside escapes, the recurring targets are virtual-disk theft and the management plane.

```bash
# which VMware product and build is in front of you?
vmware -v 2>/dev/null                              # Workstation/Fusion/ESXi version
esxcli system version get 2>/dev/null              # ESXi
curl -sk https://<host>/sdk/vimServiceVersions.xml # vCenter/ESXi API
```

## Subtopics

- **[ESXi](esxi/index.md)**: the bare-metal hypervisor and its guest-to-host escapes.
- **[vCenter](vcenter/index.md)**: the vSphere management plane.
- **[Workstation and Fusion](workstation-and-fusion/index.md)**: the desktop hypervisors.

## References

- [VMware security advisories](https://www.vmware.com/security/advisories.html)
- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware vSphere documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
