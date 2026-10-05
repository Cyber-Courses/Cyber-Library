---
title: "PV backend drivers: escaping Xen through dom0 backends"
description: "Escaping a Xen guest to dom0 through the paravirtualized backend drivers (blkback, netback) that run in dom0 and service guest frontend requests over shared rings, where flaws in request parsing corrupt dom0 kernel memory."
keywords:
  - blkback
  - netback
  - PV driver
  - dom0
  - Xen escape
---

# PV backend drivers

Paravirtualized guests use split drivers: a frontend in the guest and a backend in dom0, communicating over a shared ring. The dom0 backends (blkback for block I/O, netback for networking) parse the requests the guest places in the ring. Flaws in that parsing corrupt memory in the dom0 kernel, escaping the guest to the control domain.

```text
PV backend escape surface:
- blkback: block request ring parsing and grant mapping
- netback: packet ring handling and fragment reassembly
```

## Exploitation notes

- The backends run in the dom0 kernel, so a bug there yields dom0 kernel code execution, which controls every guest.
- They are reachable from any PV or PVHVM guest that uses paravirtualized disk or network.
- Backend request handling often pairs with the grant mechanism; see [Grant tables and event channels](grant-tables-and-event-channels.md).

## References

- [Xen security advisories](https://xenbits.xen.org/xsa/)
- [Xen Project documentation](https://xenproject.org/help/documentation/)
