---
title: "Kernel module: attacking the KVM acceleration layer"
description: "Attacking the KVM kernel module, the hardware-acceleration layer shared by every KVM-based VMM: escaping to the host kernel through flaws reachable from guest exits and the KVM ioctl interface, and through the complex nested-virtualization state machine."
keywords:
  - KVM kernel module
  - guest exit
  - KVM ioctl
  - nested virtualization
  - host kernel
---

# Kernel module

QEMU and the other VMMs emulate devices in user space, but the CPU and memory virtualization runs in the KVM kernel module. That module is a different, higher-value target than the VMM: a bug there lands in the host kernel directly, bypassing the seccomp and sVirt confinement that boxes in QEMU. The surface is small but severe, and it is shared by every KVM-based VMM, QEMU, Firecracker, Cloud Hypervisor, and the rest.

## Subtopics

- **[Kernel module escape](kernel-module-escape.md)**: host kernel code execution through the KVM module.
- **[Nested virtualization](nested-virtualization.md)**: escapes through the nested VMX and SVM state machine.

## References

- [KVM API documentation](https://www.kernel.org/doc/html/latest/virt/kvm/api.html)
- [KVM documentation](https://www.linux-kvm.org/page/Documents)
