---
title: "Nested virtualization: the VMX and SVM emulation surface"
description: "Nested virtualization lets a KVM guest run its own hypervisor, which requires KVM to emulate the hardware virtualization instructions (Intel VMX, AMD SVM) and structures (VMCS/VMCB, nested paging) in the host kernel. That emulation is complex and guest-controlled, so when nesting is enabled it is a large additional KVM-module attack surface reachable from the guest."
keywords:
  - nested virtualization
  - vmx
  - svm
  - vmcs
  - nested paging
---

# Nested virtualization

Nested virtualization allows a guest to itself be a hypervisor running sub-guests. To support it, KVM must emulate the CPU's hardware-virtualization layer: the Intel VMX or AMD SVM instructions (`VMLAUNCH`/`VMRESUME`, `VMREAD`/`VMWRITE`, `VMRUN`), the control structures (the VMCS on Intel, the VMCB on AMD), and nested paging (EPT/NPT) translation. This emulation is intricate and driven by guest-supplied structures, so enabling nesting exposes a large additional surface in the KVM kernel module, where a flaw lands in the host kernel.

## The surface

```bash
# is nesting enabled on the host? (prerequisite for this surface)
cat /sys/module/kvm_intel/parameters/nested 2>/dev/null   # Y/1 => VMX nesting on
cat /sys/module/kvm_amd/parameters/nested 2>/dev/null      # AMD SVM nesting
```

```c
// with nesting on, an attacker guest acts as an L1 hypervisor and feeds KVM:
//  - a crafted VMCS/VMCB (control and guest-state fields) that KVM parses to set up
//    the nested entry; field handling and consistency-check bugs are reachable
//  - nested EPT/NPT paging structures KVM must shadow/translate; bad entries drive
//    the translation logic into out-of-bounds or type-confusion conditions
//  - VMREAD/VMWRITE to emulated VMCS fields and nested interrupt/event injection
```

Because KVM must interpret the L1-supplied virtualization structures to run L2, the guest controls complex inputs to kernel code, and the historically productive areas are VMCS field handling and the nested paging emulation.

## Exploitation notes

- This surface exists only when nesting is enabled; it is off by default on some distributions, so check the module parameter first. When on, it substantially enlarges the [Kernel module escape](kernel-module-escape.md) surface.
- The attacker drives it by running a minimal L1 hypervisor in the guest that issues the VMX/SVM operations with crafted VMCS/VMCB and paging structures, rather than a full nested OS.
- As with other KVM-module bugs, success is host-kernel code execution, bypassing any user-space monitor sandbox entirely.
- Version-specific to the running kernel; the VMX and SVM paths are separate code, so the applicable bug depends on the host CPU vendor.

## References

- [KVM nested VMX documentation](https://docs.kernel.org/virt/kvm/x86/nested-vmx.html)
- [Intel SDM: VMX](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
- [Google Project Zero: nested virtualization research](https://googleprojectzero.blogspot.com/)
