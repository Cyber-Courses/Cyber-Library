---
title: "KVM: attacking the Linux kernel hypervisor and its products"
order: 7
description: "KVM turns the Linux kernel into a hypervisor, accelerating VMs while a user-space monitor (QEMU, Firecracker, Cloud Hypervisor) emulates devices. The attack surface spans the KVM kernel module itself, the device-emulating monitors, and the products built on KVM, QEMU-based platforms, Proxmox VE, and Nutanix AHV, each adding its own management plane."
keywords:
  - kvm
  - qemu
  - firecracker
  - proxmox
  - nutanix ahv
---

# KVM

KVM (Kernel-based Virtual Machine) is the Linux kernel's virtualization engine: it uses hardware virtualization to run guest CPUs efficiently while a user-space Virtual Machine Monitor emulates the devices. That split defines the attack surface. The KVM kernel module exposes an interface (the `/dev/kvm` ioctls and the in-kernel acceleration of instructions, MMU, and some devices) that is itself an escape and privilege-escalation surface. The monitors, QEMU, the microVM monitors Firecracker and Cloud Hypervisor, provide the device models a guest escapes through. And the products built on KVM, QEMU-based platforms, Proxmox VE, and Nutanix AHV, each wrap it with a management plane that is its own target.

```bash
# host: KVM in use and which monitor
ls -l /dev/kvm; lsmod | grep kvm
ps -ef | grep -E 'qemu|firecracker|cloud-hypervisor' | grep -v grep
```

## Subtopics

- **[QEMU](qemu/index.md)**: the dominant device emulator behind KVM.
- **[Kernel module](kernel-module/index.md)**: the KVM kernel interface and nested virtualization.
- **[microVM monitors](microvm-monitors/index.md)**: Firecracker and Cloud Hypervisor.
- **[Proxmox VE](proxmox-ve/index.md)**: the KVM-based virtualization platform.
- **[Nutanix AHV](nutanix-ahv/index.md)**: the KVM-based hyperconverged hypervisor.

## References

- [KVM documentation (kernel)](https://docs.kernel.org/virt/kvm/index.html)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM/hypervisor escape](https://github.com/WinMin/Awesome-VM-Exploit)
