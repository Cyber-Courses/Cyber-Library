---
title: "Hypercalls: escaping Xen through the guest-to-hypervisor interface"
order: 1
description: "Hypercalls are the Xen guest's direct calls into the hypervisor, covering memory management, page-table updates, I/O, and more. The hypervisor executes them in its own context with guest-supplied arguments, so a flaw in hypercall argument validation or in the sub-operations (memory, grant, mmu, physdev) corrupts or subverts the hypervisor itself, the most privileged escape target."
keywords:
  - hypercall
  - xen hypervisor
  - mmu update
  - memory op
  - argument validation
---

# Hypercalls

Hypercalls are Xen's syscall-equivalent: a guest traps into the hypervisor to request privileged operations, memory management (`mmu_update`, `mmuext_op`), memory allocation (`memory_op`), grant-table operations (`grant_table_op`), physical device access (`physdev_op`), event channels, and many more. The hypervisor runs these in its own highly privileged context using arguments the guest supplies, often pointers into guest memory it must copy and validate. A flaw in that validation, a missing bound, a type confusion between sub-operations, a page-table update that escapes the guest's allowed range, subverts the hypervisor directly, which is the most severe escape because it is below dom0 and every other domain.

## The surface

```c
// a guest issues a hypercall with a sub-command and guest-memory arguments:
//   HYPERVISOR_mmu_update(reqs, count, ...)       // batched page-table updates
//   HYPERVISOR_memory_op(cmd, &arg)               // allocate/map/exchange frames
//   HYPERVISOR_grant_table_op(cmd, uop, count)    // grant map/copy/transfer
//   HYPERVISOR_mmuext_op(...), HYPERVISOR_physdev_op(...)
// the hypervisor copies and validates these from guest memory. Primitives:
//  - an MMU update that maps a frame the guest should not own (page ownership checks)
//  - a count/index in a batched op the hypervisor trusts -> OOB over the request array
//  - type confusion between sub-op structures sharing a union
```

Page-table and memory-management hypercalls are the richest because they directly manipulate the mappings the hypervisor enforces; a bug that lets a guest map a page it does not own is a read/write primitive into other domains or the hypervisor.

## Exploitation notes

- Success lands in the hypervisor, below dom0, so it is the highest-impact Xen escape: it can reach any domain's memory and the hypervisor's own state.
- PV guests have the broadest hypercall access (they use hypercalls for page-table management directly), so classic PV configurations expose the largest surface; PVH/HVM reduce but do not remove it.
- The attacker issues crafted hypercalls from a controlled guest kernel; batched operations (arrays of requests) are a recurring locus for count/index bugs.
- Xen security advisories (XSA) catalogue hypercall bugs by sub-op; fingerprint the Xen version and match.

## References

- [Xen hypercall documentation](https://xenbits.xen.org/docs/)
- [Xen security advisories (XSA)](https://xenbits.xen.org/xsa/)
- [Google Project Zero: Xen hypercall research](https://googleprojectzero.blogspot.com/)
