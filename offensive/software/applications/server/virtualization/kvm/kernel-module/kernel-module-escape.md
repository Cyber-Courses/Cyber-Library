---
title: "Kernel module escape: host kernel code execution through KVM"
description: "Escaping to the host kernel through a vulnerability in the KVM kernel module, reached from a guest via the exits that trap into KVM and from the host via the /dev/kvm ioctl interface, which bypasses the user-space VMM confinement entirely and lands in ring 0 on the host."
keywords:
  - KVM module
  - guest exit
  - /dev/kvm
  - ioctl
  - host kernel
---

# Kernel module escape

Every guest operation that cannot run natively traps into the KVM kernel module through a VM exit: instruction emulation, MSR and control-register access, interrupt handling, and MMU events. The module also exposes the `/dev/kvm` ioctl interface to the VMM on the host. A memory-corruption or logic flaw in either path executes in the host kernel, which is the most powerful outcome in virtualization because it bypasses the VMM sandbox.

```text
KVM kernel-module surface:
- VM-exit handlers: instruction emulation, MSR/CR access, APIC, MMU
- The /dev/kvm ioctl interface used by the VMM
- The x86 emulator invoked for instructions KVM cannot execute directly
```

## Exploitation notes

- A KVM-module bug bypasses the QEMU confinement (seccomp, sVirt, non-root) entirely, because the code runs in the host kernel rather than the VMM process; this is why it is higher impact than a [QEMU device escape](../qemu/guest-to-host-escape/index.md).
- The instruction emulator is a recurring source of bugs, reached by making the CPU take an exit on a crafted instruction.
- These flaws are rarer than device-model bugs but affect every KVM-based VMM at once, since they all share the module.

## References

- [KVM API documentation](https://www.kernel.org/doc/html/latest/virt/kvm/api.html)
- [oss-security list](https://www.openwall.com/lists/oss-security/)
