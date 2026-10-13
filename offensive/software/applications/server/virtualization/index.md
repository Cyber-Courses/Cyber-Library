---
title: "Virtualization: attacking hypervisors and their management planes"
order: 9
description: "Attacking virtualization platforms: escaping a guest VM to the host through hypervisor and device-emulation flaws, taking over the management plane that controls many hosts, and stealing virtual disks and snapshots offline, across VMware, Hyper-V, KVM, Xen, Proxmox, Nutanix, and more."
keywords:
  - virtualization security
  - hypervisor
  - VM escape
  - guest to host escape
  - management plane
---

# Virtualization

A hypervisor runs untrusted guest operating systems and is supposed to keep them isolated from the host and from each other. Offensive work targets the places where that isolation leaks: the device models the hypervisor emulates for guests (the usual guest-to-host escape surface), the management plane that controls fleets of hosts, and the virtual disks and snapshots that hold guest data at rest.

The area is organized by product, because escapes are specific to each hypervisor's device-emulation code, with a consistent set of aspects under each.

## Which hypervisor am I in

From a guest, identify the hypervisor before choosing a product page; the emulated devices and signatures give it away:

```bash
# CPUID hypervisor vendor and DMI/BIOS strings are the quickest tells
systemd-detect-virt 2>/dev/null                 # kvm/vmware/microsoft/xen/oracle/parallels/bhyve
dmidecode -s system-product-name 2>/dev/null    # "VMware Virtual Platform", "VirtualBox", "KVM"...
lscpu | grep -i hypervisor; grep -o hypervisor /proc/cpuinfo | head -1
# emulated devices confirm it
lspci -nn                                        # virtio => KVM/bhyve; vmxnet/SVGA => VMware;
                                                 # Hyper-V VMBus; Parallels/VirtualBox PCI IDs
dmesg | grep -iE 'vmware|hyper-v|kvm|xen|virtualbox|parallels|bhyve'
```

The three attack aspects recur under every product: the **guest-to-host escape** through emulated devices (needs an exploit, device-model-specific), the **management plane** that controls many hosts (often just needs credentials or a reachable API), and **disk and snapshot theft** that reads guest data offline (needs host, storage, or management access, no guest exploit). Pick the aspect that matches the access you already have.

## Subtopics

- **[VMware](vmware/index.md)**: ESXi and vCenter, plus the Workstation and Fusion desktop hypervisors.
- **[Hyper-V](hyper-v/index.md)**: the Windows hypervisor and its management stack.
- **[VirtualBox](virtualbox/index.md)**: the Oracle desktop hypervisor.
- **[KVM](kvm/index.md)**: the Linux KVM ecosystem: QEMU, the kernel module, Proxmox VE, Nutanix AHV, and the Firecracker and Cloud Hypervisor microVM monitors.
- **[Xen](xen/index.md)**: the Xen hypervisor and its control domain.
- **[Parallels Desktop](parallels-desktop/index.md)**: the macOS desktop hypervisor.
- **[bhyve](bhyve/index.md)**: the FreeBSD hypervisor.

## References

- [Pwn2Own results archive](https://www.zerodayinitiative.com/blog)
- [QEMU security process](https://www.qemu.org/docs/master/system/security.html)
