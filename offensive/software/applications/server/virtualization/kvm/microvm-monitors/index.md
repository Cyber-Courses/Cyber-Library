---
title: "microVM monitors: attacking Firecracker and Cloud Hypervisor"
description: "Firecracker and Cloud Hypervisor are minimal KVM device monitors built for serverless and container-isolation workloads. They deliberately expose a tiny device set, virtio over MMIO, a serial console, and little else, to shrink the escape surface. Attacking them means targeting that reduced virtio and API surface and the host-side orchestration that manages many microVMs."
keywords:
  - firecracker
  - cloud hypervisor
  - microvm
  - virtio-mmio
  - minimal device model
---

# microVM monitors

Firecracker (from AWS) and Cloud Hypervisor are modern, minimal Virtual Machine Monitors for KVM, written in Rust and designed for serverless and container-isolation use where density and a small attack surface matter more than device breadth. They deliberately emulate almost nothing: a handful of virtio devices over MMIO (block, net, vsock, and a few others), a serial console, and a minimal interrupt controller, with no legacy PCI, no USB, no graphics, no audio. That shrinks the guest-to-host escape surface to the virtio implementations and the monitor's own API, and the Rust implementation removes most classic memory-safety bugs, so attacks focus on logic flaws, `unsafe` blocks, and the host-side orchestration.

```bash
# identify the monitor on the host
ps -ef | grep -E 'firecracker|cloud-hypervisor' | grep -v grep
# firecracker is controlled by a REST API on a unix socket
ls -l /run/firecracker*.socket 2>/dev/null
```

## Subtopics

- **[Firecracker](firecracker.md)**: the AWS microVM monitor and its reduced surface.
- **[Cloud Hypervisor](cloud-hypervisor.md)**: the Rust VMM and its device and API surface.

## References

- [Firecracker design and security](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md)
- [Cloud Hypervisor](https://www.cloudhypervisor.org/)
- [rust-vmm (shared virtio/VMM components)](https://github.com/rust-vmm)
