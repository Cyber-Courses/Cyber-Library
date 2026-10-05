---
title: "bhyve: attacking the FreeBSD hypervisor"
description: "bhyve is FreeBSD's type-2 hypervisor, using hardware virtualization with device models emulated in a userspace bhyve process. The guest-to-host escape targets those device models, virtio, AHCI, e1000, and the USB and framebuffer devices, with a flaw landing in the bhyve process on the host, alongside theft of the guest disk images."
keywords:
  - bhyve
  - freebsd
  - vmm
  - virtio
  - device model
---

# bhyve

bhyve is FreeBSD's native hypervisor. The `vmm.ko` kernel module provides CPU and memory virtualization using VT-x or AMD-V, while a per-VM userspace `bhyve` process emulates the devices. The design is lean and modern, with a focus on virtio and a small set of emulated devices (AHCI storage, the e1000 NIC, USB, a framebuffer, and PCI passthrough). A guest-to-host escape therefore targets the device models in the `bhyve` process, landing on the FreeBSD host, and the usual disk-image theft applies to the guest's backing storage.

```bash
# on a FreeBSD bhyve host
kldstat | grep vmm; ls /dev/vmm/
ps aux | grep bhyve | grep -v grep
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape.md)**: breaking out through bhyve device models.
- **[Host access and shell](host-access-and-shell.md)**: execution on the FreeBSD host.
- **[Known escape exploits](known-escape-exploits.md)**: the recurring device-model bugs.

## References

- [FreeBSD Handbook: bhyve](https://docs.freebsd.org/en/books/handbook/virtualization/#virtualization-host-bhyve)
- [bhyve(8) manual](https://man.freebsd.org/cgi/man.cgi?query=bhyve)
- [FreeBSD security advisories](https://www.freebsd.org/security/advisories/)
