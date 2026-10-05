---
title: "Host access and shell: execution on the AHV node and Controller VM"
description: "An AHV deployment has two shells worth reaching: the AHV hypervisor node itself and the Controller VM that runs the storage fabric and management services. Access comes from a guest escape, node or CVM credentials, or SSH, and the Controller VM is the higher-value target because it mediates storage and control for the node and connects to the cluster."
keywords:
  - ahv host
  - controller vm
  - cvm
  - ssh
  - nutanix
---

# Host access and shell

AHV presents two host-level targets. The AHV node is a Linux hypervisor running the QEMU processes; the Controller VM (CVM) is a VM on that node that provides the distributed storage and hosts the Nutanix services and the Prism interface. Reaching a shell on either comes from a guest escape landing on the node, from node or CVM credentials, or from SSH. The CVM is the higher-value target: it mediates all storage I/O for the node, holds service credentials and keys, and is networked to the other CVMs in the cluster, so CVM access scales toward cluster control.

## Reach the shells

```bash
# AHV node (hypervisor) via SSH with node credentials
ssh root@<ahv-node>
virsh list --all                          # VMs on this node, including the CVM
# the CVM, reachable on the internal network from the node (commonly 192.168.5.2)
ssh nutanix@<cvm-ip>
# Nutanix CLIs run on the CVM
acli vm.list; ncli cluster info 2>/dev/null
```

## What CVM access gives

```bash
# the CVM controls storage and management for the node and talks to peer CVMs
# enumerate the cluster and its storage containers (vdisks for every VM)
acli vm.list; acli image.list
ncli storagepool ls; ncli container ls
# service credentials and keys on the CVM pivot to Prism and other CVMs
```

## Exploitation notes

- Prioritise the Controller VM over the bare node: it mediates storage (so it reaches every VM's vdisks), runs the management services, and is meshed with peer CVMs, making it the lever on the whole cluster.
- The CVM sits on an internal node-to-CVM network (often 192.168.5.0/24) reachable from the AHV node, so a node foothold from a guest escape leads naturally to the CVM.
- The `acli` and `ncli` tools on the CVM enumerate and control VMs, images, and storage; use them to map the cluster and locate high-value VMs and their disks.
- Persistence on the node or CVM follows standard Linux mechanisms; the storage and management reach is what makes CVM access distinctive.

## References

- [Nutanix AHV administration](https://portal.nutanix.com/)
- [Nutanix CLI (acli/ncli) reference](https://portal.nutanix.com/)
