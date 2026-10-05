---
title: "Host access and shell: reaching the AHV host and Controller VM"
description: "Reaching a Nutanix node: the AHV hypervisor host and the Controller VM (CVM) that provides storage and runs the management services, through SSH to the nutanix user and the acli and acropolis interfaces, from where guests and the storage fabric are controlled."
keywords:
  - Nutanix CVM
  - AHV host
  - acli
  - nutanix user
  - host access
---

# Host access and shell

A Nutanix node has two shells that matter: the AHV host (a CentOS/EL-based KVM host) and the Controller VM (CVM), a VM on each node that provides the distributed storage and runs cluster services. The CVM's `nutanix` account is the operational identity, with `acli` and cluster tooling that control VMs and storage. Reaching either, by SSH credentials, spraying, or a management pivot, is broad control.

```bash
# On the CVM (nutanix user)
acli vm.list
acli vm.get <vm>
ncli cluster info
# On the AHV host: it is KVM underneath
virsh list --all
```

## Exploitation notes

- The CVM `nutanix` account controls VMs and storage cluster-wide, and CVMs trust each other, so one CVM often reaches the others.
- The AHV host is a KVM host, so host access exposes the QEMU processes and the local guest definitions.
- CVM and host credentials are frequently shared or weakly rotated across a cluster; one node's access tends to generalize.

## References

- [Nutanix: AHV and the Controller VM](https://www.nutanix.dev/)
- [Nutanix command reference (acli, ncli)](https://portal.nutanix.com/)
