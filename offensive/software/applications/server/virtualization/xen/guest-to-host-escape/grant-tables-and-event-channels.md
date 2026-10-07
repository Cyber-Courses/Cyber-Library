---
title: "Grant tables and event channels: escaping Xen through the inter-domain machinery"
order: 2
description: "Grant tables let a Xen domain share its memory pages with another, and event channels deliver inter-domain signals. The paravirtual drivers rely on both. Flaws in grant map/copy/transfer handling, in reference counting on grant teardown, and in event-channel allocation give cross-domain memory access or hypervisor corruption reachable from a guest."
keywords:
  - grant tables
  - event channels
  - grant reference
  - shared memory
  - inter-domain
---

# Grant tables and event channels

Xen's split-driver model needs domains to share memory and signal each other, and two mechanisms provide it. Grant tables let a domain grant another domain access (map or copy) to specific pages, identified by grant references; this is how a guest exposes its I/O ring buffers and data pages to the dom0 backend. Event channels are the lightweight inter-domain signal (an interrupt-like notification). Both are managed through hypercalls and tracked by the hypervisor, so flaws in grant map/copy/transfer handling, in the reference counting that governs when a granted page can be reclaimed, and in event-channel allocation and binding, yield cross-domain memory access or hypervisor state corruption from a guest.

## The surface

```c
// grant operations (via HYPERVISOR_grant_table_op):
//   GNTTABOP_map_grant_ref   - map a page another domain granted
//   GNTTABOP_copy            - copy to/from a granted page
//   GNTTABOP_transfer        - transfer page ownership
// Primitives:
//  - a grant/copy with a length or offset the handler trusts -> OOB access
//  - reference-count errors on map/unmap letting a page be reused while still mapped
//    (a use-after-free across domains, or a guest retaining access after teardown)
//  - event-channel allocation/binding flaws (exhaustion, confusion of port types)
```

The reference-counting angle is historically important: if a guest can keep a grant mapped after the owner believes it reclaimed the page, it retains access to memory the hypervisor has reassigned, bridging into dom0 or another domain.

## Exploitation notes

- Grant copy/map length and offset handling is the direct OOB surface; the reference-counting lifecycle (map, use, unmap, reclaim) is the subtler and recurrently buggy one, producing cross-domain use-after-free.
- These mechanisms underlie the PV backends, so they are reachable by any guest using paravirtual block or network; the attacker drives them from a controlled frontend/guest kernel.
- Impact ranges from cross-domain memory read/write (reaching dom0 or peers) to hypervisor corruption, depending on the bug; both are severe.
- Catalogued in XSAs; match the Xen version. The backends that use these are covered in [PV backend drivers](pv-backend-drivers.md).

## References

- [Xen grant tables documentation](https://xenbits.xen.org/docs/)
- [Xen security advisories (XSA)](https://xenbits.xen.org/xsa/)
