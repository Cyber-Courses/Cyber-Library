---
title: "Guest to host escape: breaking out of a Xen guest"
description: "Escaping a Xen guest to dom0 or the hypervisor through the interfaces a guest can reach: hypercalls into the hypervisor, the grant-table and event-channel mechanisms, and the paravirtualized backend drivers in dom0. HVM guests additionally reach the QEMU device models."
keywords:
  - Xen escape
  - hypercall
  - grant table
  - backend driver
  - guest to host
---

# Guest to host escape

A Xen guest reaches the host through several interfaces, each an escape surface with a different impact. Hypercalls enter the hypervisor directly; grant tables and event channels mediate shared memory with dom0; paravirtualized guests talk to backend drivers in dom0. HVM guests additionally use a QEMU device-model process, inheriting the [QEMU device surface](../../kvm/qemu/guest-to-host-escape/index.md).

## Subtopics

- **[Hypercalls](hypercalls.md)**: the direct guest-to-hypervisor interface.
- **[Grant tables and event channels](grant-tables-and-event-channels.md)**: shared memory and signalling with dom0.
- **[PV backend drivers](pv-backend-drivers.md)**: blkback and netback in dom0.

## References

- [Xen Project documentation](https://xenproject.org/help/documentation/)
- [Xen security advisories](https://xenbits.xen.org/xsa/)
