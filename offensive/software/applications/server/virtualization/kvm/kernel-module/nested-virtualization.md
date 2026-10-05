---
title: "Nested virtualization: escaping through the KVM nested state machine"
description: "Abusing KVM nested virtualization, where the KVM module emulates Intel VMX or AMD SVM so a guest can itself run a hypervisor, exercising the complex nested-state handling (VMCS and VMCB shadowing) that has produced host kernel escapes from a nested guest."
keywords:
  - nested virtualization
  - VMX
  - SVM
  - VMCS shadowing
  - KVM
---

# Nested virtualization

Nested virtualization lets a KVM guest run its own hypervisor by having the KVM module emulate the hardware virtualization extensions (Intel VMX, AMD SVM) for that guest. Emulating VMX or SVM means shadowing the control structures (VMCS, VMCB) and handling nested exits, a large and intricate state machine. Flaws there let a nested guest corrupt host kernel state, escaping both its own hypervisor and the host.

```text
Nested-virtualization surface:
- VMCS / VMCB shadowing and consistency checks
- Nested VM-exit and VM-entry handling
- Emulation of the virtualization instructions (VMREAD/VMWRITE, VMRUN)
```

## Exploitation notes

- Nested virtualization must be enabled (`kvm_intel.nested` / `kvm_amd.nested`), so its reachability depends on the host configuration; cloud instances that offer nested virt expose it.
- The attacker runs a hypervisor inside the guest to drive the nested paths, reaching code that non-nested guests never touch.
- Like a direct [Kernel module escape](kernel-module-escape.md), success lands in the host kernel, below the VMM sandbox.

## References

- [KVM: nested virtualization](https://www.kernel.org/doc/html/latest/virt/kvm/nested-vmx.html)
- [KVM API documentation](https://www.kernel.org/doc/html/latest/virt/kvm/api.html)
