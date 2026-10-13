---
title: "Guest-to-host escape: breaking out of a Xen domU"
order: 2
description: "A Xen guest escapes toward the hypervisor or the privileged dom0 through three surfaces: the hypercall interface the guest calls into the hypervisor, the grant-table and event-channel mechanisms that share memory and signals between domains, and the paravirtual backend drivers in dom0 that parse guest ring requests. Each is guest-reachable and has produced breakouts."
keywords:
  - xen escape
  - hypercall
  - grant tables
  - event channels
  - pv backend
---

# Guest-to-host escape

A Xen domU reaches outside itself through well-defined interfaces, and each is an escape surface. Hypercalls are the guest's direct calls into the hypervisor, so a flaw in hypercall handling corrupts or subverts the hypervisor itself. Grant tables and event channels are how domains share memory pages and signal each other; mismanagement there bridges a guest into dom0 or hypervisor memory. And the paravirtual backend drivers (block, network) running in dom0 parse the ring requests a guest frontend posts, so a backend bug gives code execution in the privileged dom0. The target is whichever of these the configuration exposes, PV, PVH, and HVM guests reach somewhat different mixes.

```bash
# the PV interfaces visible in a Xen guest
ls /sys/bus/xen-backend 2>/dev/null; ls /dev/xen 2>/dev/null
cat /proc/xen/capabilities 2>/dev/null
dmesg | grep -iE 'grant|event channel|xenbus'
```

## Subtopics

- **[Hypercalls](hypercalls.md)**: the guest-to-hypervisor call interface.
- **[Grant tables and event channels](grant-tables-and-event-channels.md)**: the inter-domain memory and signal machinery.
- **[PV backend drivers](pv-backend-drivers.md)**: the dom0 backend halves of paravirtual devices.

## References

- [Xen hypercall interface](https://xenbits.xen.org/docs/)
- [Xen security advisories (XSA)](https://xenbits.xen.org/xsa/)
