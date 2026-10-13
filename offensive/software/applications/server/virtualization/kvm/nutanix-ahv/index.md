---
title: "Nutanix AHV: attacking the hyperconverged KVM hypervisor"
order: 4
description: "Nutanix AHV is a KVM/QEMU-based hypervisor in a hyperconverged platform where a Controller VM on each node runs the storage fabric and the management services. The guest-to-host escape inherits QEMU's device models; the distinctive surface is the Controller VM, the Prism management plane, and the distributed storage holding every VM's virtual disks."
keywords:
  - nutanix ahv
  - controller vm
  - prism
  - kvm
  - hyperconverged
---

# Nutanix AHV

Nutanix AHV is the company's native hypervisor, built on KVM with QEMU-derived device emulation. What distinguishes it is the hyperconverged architecture: each node runs a Controller VM (CVM) that provides the distributed storage fabric and hosts the management services, with the Prism interface (per-cluster Element and multi-cluster Central) on top. Offensively, the guest-to-host escape is the familiar QEMU device surface, but the high-value targets are the Controller VM, which mediates storage and management, the Prism API, and the distributed storage that holds every VM's virtual disks.

```bash
# on an AHV host: KVM/QEMU plus the Nutanix stack
ps -ef | grep -E 'qemu|acropolis' | grep -v grep
# the Controller VM runs the storage and management services
virsh list --all 2>/dev/null | grep -i cvm
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape.md)**: breaking out of an AHV VM via QEMU device models.
- **[Host access and shell](host-access-and-shell.md)**: execution on the AHV host and the Controller VM.
- **[Management plane](management-plane.md)**: the Prism API and Controller VM services.
- **[Disk and snapshot theft](disk-and-snapshot-theft.md)**: taking vdisks from the storage fabric.
- **[Known escape exploits](known-escape-exploits.md)**: the inherited QEMU and Nutanix-specific bugs.

## References

- [Nutanix AHV documentation](https://www.nutanix.com/products/ahv)
- [Nutanix security advisories](https://www.nutanix.com/support-services/security-advisories)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
