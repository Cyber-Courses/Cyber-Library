---
title: "microVM monitors: attacking Firecracker and Cloud Hypervisor"
description: "Attacking lightweight virtual machine monitors used for microVMs: Firecracker and Cloud Hypervisor. Their deliberately minimal device models shrink the guest-to-host escape surface but do not remove it, and they also back sandboxed container runtimes."
keywords:
  - microVM
  - Firecracker
  - Cloud Hypervisor
  - VMM
  - VM escape
---

# microVM monitors

microVM monitors are minimal virtual machine monitors built for fast, dense, isolated workloads, especially serverless and sandboxed containers. Firecracker and Cloud Hypervisor expose only a handful of virtio devices and a thin API, which shrinks the guest-to-host escape surface compared to full QEMU, but the remaining device models and the management API are still attackable. These monitors also back sandboxed container runtimes.

## Subtopics

- **[Firecracker](firecracker.md)**: the AWS microVM monitor.
- **[Cloud Hypervisor](cloud-hypervisor.md)**: the Rust VMM.

## References

- [Firecracker design](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md)
- [Cloud Hypervisor](https://www.cloudhypervisor.org/)
