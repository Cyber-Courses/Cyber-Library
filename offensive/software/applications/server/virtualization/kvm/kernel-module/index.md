---
title: "Kernel module: attacking the KVM kernel interface"
order: 2
description: "The KVM kernel module accelerates guest execution and exposes the /dev/kvm ioctl interface and in-kernel device emulation (such as the local APIC and the coalesced MMIO path). Bugs here run in the host kernel rather than a user-space monitor, so a KVM module flaw is a direct host-kernel compromise, and nested virtualization expands the exposed emulation surface."
keywords:
  - kvm module
  - /dev/kvm
  - in-kernel emulation
  - nested virtualization
  - host kernel
---

# Kernel module

Most KVM escapes target the user-space monitor, but the KVM kernel module itself is a surface, and a far more severe one, because its code runs in the host kernel. KVM accelerates guest instruction, MMU, and interrupt handling, and emulates a few devices in-kernel for speed (the local APIC, the I/O APIC and PIT, the coalesced MMIO ring, and MSR/CPUID handling). A guest reaches this code through the operations KVM accelerates, and a memory-safety or logic flaw in the module is host-kernel code execution, bypassing any monitor sandbox entirely. Nested virtualization, where a guest runs its own hypervisor, adds the VMX/SVM emulation paths and greatly widens this surface.

```bash
# in-kernel emulation and nested support
lsmod | grep -E 'kvm_intel|kvm_amd'
cat /sys/module/kvm_intel/parameters/nested 2>/dev/null   # nested VMX enabled?
cat /sys/module/kvm_amd/parameters/nested 2>/dev/null
```

## Subtopics

- **[Kernel module escape](kernel-module-escape.md)**: flaws in the in-kernel emulation reachable from a guest.
- **[Nested virtualization](nested-virtualization.md)**: the VMX/SVM emulation surface a nested guest exposes.

## References

- [KVM API documentation](https://docs.kernel.org/virt/kvm/api.html)
- [Google Project Zero: KVM research](https://googleprojectzero.blogspot.com/)
- [KVM nested virtualization](https://docs.kernel.org/virt/kvm/x86/nested-vmx.html)
