---
title: "Guest to host escape: breaking out of a bhyve guest"
description: "Escaping a bhyve guest to the FreeBSD host by exploiting its device models: the virtio devices, the e1000 network adapter, the framebuffer, and the USB and block emulation, which run in the host bhyve process and parse guest-controlled input."
keywords:
  - bhyve escape
  - device model
  - virtio
  - e1000
  - guest to host
---

# Guest to host escape

Each bhyve guest is served by a `bhyve` process on the host that emulates its devices in user space, much like QEMU. Those device models parse guest-controlled input, so memory-corruption flaws in the virtio devices, the e1000 network adapter, the framebuffer, or the USB and block emulation let a guest execute code in the host bhyve process.

```text
bhyve guest escape surfaces (reachable from a guest):
- virtio devices (net, block, console)
- e1000 network adapter
- The framebuffer / display
- USB and AHCI/block emulation
```

## Exploitation notes

- Code execution lands in the host `bhyve` process; FreeBSD mitigations and Capsicum capability-mode confinement, where used, limit the outcome, so check the host's confinement.
- The device set is configured per VM, so reachable surface depends on which emulated devices the guest has.
- Named instances are under [Known escape exploits](known-escape-exploits.md).

## References

- [bhyve man page](https://man.freebsd.org/cgi/man.cgi?bhyve)
- [FreeBSD security advisories](https://www.freebsd.org/security/advisories/)
