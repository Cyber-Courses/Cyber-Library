---
title: "bhyve: attacking the FreeBSD hypervisor"
description: "Attacking bhyve, the FreeBSD hypervisor: reaching the host that runs it, and escaping a guest to the host through bhyve's device models, which emulate guest hardware in a host user-space process much as QEMU does."
keywords:
  - bhyve
  - FreeBSD
  - hypervisor
  - device model
  - VM escape
---

# bhyve

bhyve is the native FreeBSD hypervisor, also used in appliances and some cloud and storage products built on FreeBSD. It emulates guest devices in a host user-space process, so its attack model mirrors QEMU's: reach the FreeBSD host that controls the guests, and escape a guest through flaws in the emulated device models.

## Subtopics

- **[Host access and shell](host-access-and-shell.md)**: reaching the FreeBSD host.
- **[Guest to host escape](guest-to-host-escape.md)**: breaking out through device models.
- **[Known escape exploits](known-escape-exploits.md)**: named bhyve breakouts.

## References

- [FreeBSD Handbook: bhyve](https://docs.freebsd.org/en/books/handbook/virtualization/#virtualization-host-bhyve)
- [bhyve man page](https://man.freebsd.org/cgi/man.cgi?bhyve)
