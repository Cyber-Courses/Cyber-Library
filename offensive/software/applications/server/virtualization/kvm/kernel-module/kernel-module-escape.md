---
title: "Kernel module escape: host-kernel compromise through KVM in-kernel emulation"
description: "KVM emulates some devices and instructions inside the host kernel for performance: the local APIC, I/O APIC, PIT, coalesced MMIO, and MSR/CPUID handling. A guest reaches these paths through MMIO, I/O, and privileged instructions, so a memory-safety or logic flaw there executes in the host kernel directly, a more powerful escape than corrupting a user-space monitor."
keywords:
  - kvm escape
  - in-kernel emulation
  - local apic
  - coalesced mmio
  - host kernel
---

# Kernel module escape

The KVM module emulates a performance-critical subset of the platform in the host kernel, and that code is reachable from the guest. The in-kernel local APIC and I/O APIC, the PIT timer, the coalesced MMIO ring buffer, and the handling of certain MSRs, CPUID, and instruction emulation all process guest-triggered events in kernel context. A flaw there, an out-of-bounds access in APIC register handling, a bad index in the coalesced MMIO ring, a logic error in instruction emulation, is host-kernel code execution or memory corruption, which is strictly more powerful than a user-space monitor escape because it skips the monitor sandbox and lands in ring 0 on the host.

## The in-kernel surface

```c
// a guest reaches in-kernel KVM code by the operations KVM handles without exiting
// to the monitor:
//  - APIC: MMIO/x2APIC MSR accesses to the emulated local APIC register page
//  - coalesced MMIO: writes to registered regions fill an in-kernel ring the guest
//    and host share; a bad first/last index is a classic OOB primitive
//  - instruction emulation: KVM's x86 emulator decodes and executes some guest
//    instructions in-kernel; operand/memory handling bugs are reachable here
//  - MSR/CPUID: handlers for guest reads/writes of model-specific registers
```

The historically productive spots are the coalesced MMIO ring index handling and the x86 instruction emulator, both of which process rich guest-controlled input in the kernel.

## Exploitation notes

- Landing in the host kernel is the distinguishing feature: unlike a QEMU escape, there is no user-space sandbox to then break out of, so a KVM-module bug is a direct ring-0 host compromise.
- The coalesced MMIO ring is a shared guest/host structure with indices the guest can influence, making index-bounding bugs a recurring OOB source; the instruction emulator is a complex decoder reachable for specific instructions.
- Reaching these paths needs the guest to perform the triggering MMIO/MSR/instruction, which an attacker-controlled guest kernel does directly.
- Version-specific to the running kernel; fingerprint `uname -r` and the kvm module, and note nested virtualization widens this surface, see [Nested virtualization](nested-virtualization.md).

## References

- [KVM API and in-kernel devices](https://docs.kernel.org/virt/kvm/api.html)
- [Google Project Zero: KVM in-kernel emulation research](https://googleprojectzero.blogspot.com/)
- [Awesome VM escape: KVM](https://github.com/WinMin/Awesome-VM-Exploit)
