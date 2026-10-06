---
title: "PV backend drivers: escaping Xen through the dom0 device backends"
order: 3
description: "Xen paravirtual devices are split: a frontend in the guest and a backend in dom0 (or a driver domain) communicate over a shared-memory ring. The backends (blkback, netback, and others) parse guest-posted ring requests in the privileged dom0, so a flaw in request parsing or grant handling gives code execution in dom0, which controls every domain."
keywords:
  - pv backend
  - blkback
  - netback
  - ring buffer
  - dom0
---

# PV backend drivers

Xen's paravirtual I/O uses a split-driver model: the guest runs a frontend (`blkfront`, `netfront`) that posts requests into a shared-memory ring, and the backend half (`blkback`, `netback`, and others) runs in dom0 or a dedicated driver domain and services them. The backend reads the guest-posted ring requests, follows grant references to the guest's data pages, and performs the I/O. Because that parsing happens in the privileged dom0, a flaw in how a backend validates ring requests, request counts, segment descriptors, or the grant references they carry, gives code execution in dom0, which administers every domain.

## The surface

```c
// a guest frontend posts requests into the shared ring; the dom0 backend consumes:
//   blkback: block requests with segment descriptors (grant ref + offset + length)
//            per segment; a segment count/length the backend trusts, or a grant ref
//            it maps without proper validation, -> OOB or cross-domain access in dom0
//   netback: packet buffers described by grant refs and lengths; offload/fragment
//            handling with trusted lengths is a classic locus
// ring indices (req_prod/req_cons) are in shared memory; a backend that trusts them
// without bounding against the ring size is another primitive
```

The block backend's multi-segment request format and the network backend's fragment and offload handling are the recurring loci, both parsing guest-chosen counts and lengths in dom0.

## Exploitation notes

- Code execution lands in dom0 (or a driver domain), which is the privileged control domain, so a backend bug is effectively host compromise, controlling all guests, their disks, and the toolstack.
- The backends consume grant references for the data pages, so these bugs often intertwine with the [grant tables](grant-tables-and-event-channels.md) surface; a grant or ring mishandling is the shared theme.
- Driver domains (running backends in a dedicated, less-privileged domain) contain the impact to that domain rather than full dom0 where configured; check whether backends run in dom0 or isolated driver domains.
- The attacker drives this from a controlled guest frontend posting crafted ring requests; match the Xen/dom0 kernel version to the backend advisory.

## References

- [Xen PV drivers and split model](https://wiki.xenproject.org/wiki/Paravirtualization_(PV))
- [Xen security advisories (XSA)](https://xenbits.xen.org/xsa/)
