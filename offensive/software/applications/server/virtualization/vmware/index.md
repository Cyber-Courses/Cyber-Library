---
title: "VMware: attacking ESXi, vCenter, and the desktop hypervisors"
description: "Attacking VMware virtualization: the ESXi hypervisor host, the vCenter management plane that controls many ESXi hosts, and the Workstation and Fusion desktop hypervisors. Covers host access, guest-to-host escapes, SSO and token abuse, and virtual disk theft."
keywords:
  - VMware
  - ESXi
  - vCenter
  - vSphere
  - VM escape
---

# VMware

VMware vSphere is the enterprise virtualization benchmark: ESXi hosts run the VMs, and vCenter manages fleets of them. Offensively they are different targets. ESXi is a hardened appliance reached through its shell and API, with guest-to-host escapes in its device emulation. vCenter is a web and API appliance whose SSO and management flaws grant control of every host it manages. The Workstation and Fusion desktop products share much of ESXi's device-emulation code.

## Subtopics

- **[ESXi](esxi/index.md)**: the hypervisor host.
- **[vCenter](vcenter/index.md)**: the management plane.
- **[Workstation and Fusion](workstation-and-fusion/index.md)**: the desktop hypervisors.

## References

- [VMware vSphere security](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
