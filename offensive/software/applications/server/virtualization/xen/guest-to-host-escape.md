---
title: "Guest to host escape: breaking out of a Xen guest"
description: "Escaping a Xen guest to dom0 or the hypervisor through the interfaces a guest can reach: hypercalls into the hypervisor, the grant-table and event-channel mechanisms, the paravirtualized backend drivers in dom0, and the QEMU device models used for HVM guests."
keywords:
  - Xen escape
  - hypercall
  - grant table
  - backend driver
  - guest to host
---

# Guest to host escape

A Xen guest reaches the host through several interfaces, each an escape surface. Hypercalls enter the hypervisor directly, so a flaw there yields hypervisor code execution. Grant tables and event channels mediate shared memory with dom0. Paravirtualized guests talk to backend drivers in dom0 (blkback, netback), and HVM guests additionally use a QEMU device-model process, inheriting QEMU's device surface.

```text
Xen escape surfaces reachable from a guest:
- Hypercalls into the hypervisor (highest impact)
- Grant tables and event channels (shared memory with dom0)
- PV backend drivers in dom0: blkback, netback
- QEMU device models for HVM guests (see KVM/QEMU)
```

## Exploitation notes

- Hypercall and grant-table flaws land in the hypervisor or dom0 kernel, the most powerful outcome; backend-driver bugs land in dom0.
- HVM guests add the full QEMU device surface, so the [KVM and QEMU guest to host escape](../kvm/qemu/guest-to-host-escape.md) techniques apply to Xen HVM.
- Named instances tracked as Xen Security Advisories are under [Known escape exploits](known-escape-exploits.md).

## References

- [Xen Project: hypercall interface](https://xenproject.org/help/documentation/)
- [Xen security advisories](https://xenbits.xen.org/xsa/)
