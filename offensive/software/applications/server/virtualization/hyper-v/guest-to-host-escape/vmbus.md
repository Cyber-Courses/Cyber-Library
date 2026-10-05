---
title: "VMBus: escaping Hyper-V through the channel transport"
description: "VMBus is the ring-buffer transport that carries all synthetic-device traffic between a Hyper-V guest and the root partition. The guest controls channel setup, GPADL memory registration, and the packets placed in the ring. Flaws in how the root-partition endpoints parse ring packets, channel offers, and GPADL descriptors give memory corruption in the privileged root partition."
keywords:
  - vmbus
  - ring buffer
  - gpadl
  - channel
  - root partition
---

# VMBus

VMBus is the backbone of Hyper-V's paravirtual I/O: every synthetic device communicates with its Virtualization Service Provider in the root partition over a VMBus channel, which is a pair of ring buffers in guest-shared memory. The guest participates in channel negotiation, registers memory with the root partition through GPADL (Guest Physical Address Descriptor List) descriptors, and writes request packets into the ring. All of that is guest-controlled data parsed by root-partition code, so the transport itself, before any device logic, is an escape surface: malformed ring packets, channel offers, or GPADL descriptors can drive the root-partition endpoint out of bounds.

## The transport surface

```c
// a VMBus packet has a descriptor header (type, offset, length, flags) followed by
// payload; the receiver uses the header's offset/length to locate the payload in
// the ring. A length or offset the root-partition reader trusts, or a ring index
// it advances without bounds checking, yields OOB read/write in the root partition.

// GPADL registers guest memory for a channel: the guest supplies a descriptor list
// of guest physical page ranges. A crafted GPADL (bad ranges, inconsistent counts)
// that the root partition maps/walks without validation is a second primitive.
```

The ring-buffer indices (read and write pointers) are in shared memory the guest can influence, so a classic bug is a receiver that computes a packet length or next-index from guest-controlled fields and reads past the ring or into unintended memory.

## Exploitation notes

- VMBus is below the device logic, so a transport-level bug can affect multiple synthetic devices at once; it is reached as soon as a guest can open and drive a channel, which any paravirtual device does.
- GPADL handling is a distinct, high-value surface: it is where guest physical memory is registered with the root partition, and mismanagement there bridges guest-controlled addressing into host mappings.
- The root partition endpoints run with high privilege (kernel VSPs or the worker process), so corruption there is close to full host control; the specific endpoint depends on which channel is abused.
- Build-specific, as always; fingerprint the Windows/Hyper-V version and match to the VMBus or VSP advisory.

## References

- [Microsoft: VMBus and synthetic device architecture](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [MSRC: Hyper-V VMBus research](https://www.microsoft.com/en-us/msrc)
- [Hyper-V internals (community research)](https://github.com/microsoft/MSRC-Security-Research)
