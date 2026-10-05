---
title: "QEMU: attacking the device emulator behind KVM"
description: "QEMU provides the device emulation for KVM virtual machines and underlies most Linux virtualization products. Its offensive surface is the guest-to-host escape through device models, the QMP/HMP monitor control interface, theft of qcow2 disk images and snapshots, and obtaining a shell on the host QEMU runs on. The device-model bugs recur across every KVM-based product."
keywords:
  - qemu
  - kvm
  - device model
  - qmp
  - qcow2
---

# QEMU

QEMU is the user-space device emulator that pairs with the KVM kernel module to run virtual machines, and it underlies most Linux virtualization: libvirt, Proxmox, OpenStack, oVirt, cloud platforms, and more. Its offensive surface is therefore the common denominator of KVM-based products. The central concern is the guest-to-host escape through QEMU's device models; alongside it are the QMP/HMP monitor that controls a running VM, the qcow2 disk images and snapshots that hold guest data, and the host QEMU runs on. Because every KVM product reuses these device models, a QEMU escape bug tends to apply everywhere.

```bash
# from the guest: the emulated device inventory (escape surface)
lspci -nn; lsusb; cat /proc/ioports | head
# from the host: QEMU processes and their monitor sockets
ps aux | grep qemu; ls -l /var/lib/libvirt/qemu/*.monitor 2>/dev/null
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape/index.md)**: breaking out through QEMU device models.
- **[Management plane](management-plane.md)**: the QMP/HMP monitor and libvirt control.
- **[Disk and snapshot theft](disk-and-snapshot-theft.md)**: stealing qcow2 images and snapshots.
- **[Host access and shell](host-access-and-shell.md)**: execution on the QEMU host.
- **[Known escape exploits](known-escape-exploits.md)**: the recurring device-model escapes.

## References

- [QEMU documentation](https://www.qemu.org/docs/master/)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM/hypervisor escape research](https://github.com/WinMin/Awesome-VM-Exploit)
