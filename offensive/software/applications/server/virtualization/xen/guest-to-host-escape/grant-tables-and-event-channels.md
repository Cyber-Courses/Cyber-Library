---
title: "Grant tables and event channels: escaping Xen through shared memory"
description: "Escaping Xen through the grant-table and event-channel mechanisms that let a guest share memory pages with and signal dom0, where reference-counting, page-type, and mapping errors corrupt host state and can escape to dom0 or the hypervisor."
keywords:
  - grant table
  - event channel
  - shared memory
  - page type
  - Xen escape
---

# Grant tables and event channels

Grant tables let a guest grant another domain (usually dom0) access to its memory pages, and event channels deliver the asynchronous signals that drive paravirtualized I/O. The hypervisor tracks grant references, page types, and mappings, and errors in that bookkeeping (double-grant, type confusion, reference-count mishandling) corrupt host memory and can escape to dom0 or the hypervisor.

```text
Grant / event-channel escape surface:
- Grant reference tracking, mapping, and page-type transitions
- Reference counting on granted pages
- Event-channel allocation and delivery
```

## Exploitation notes

- These mechanisms underpin all PV and PVHVM I/O, so they are reachable from essentially any guest.
- The bug classes are reference-count and page-type errors, consistent with the Xen Security Advisory record.
- Impact ranges from dom0 memory corruption to hypervisor compromise depending on the flaw.

## References

- [Xen security advisories](https://xenbits.xen.org/xsa/)
- [Xen Project: grant tables](https://xenbits.xen.org/docs/)
