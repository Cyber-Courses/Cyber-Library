---
title: "virtio devices: escaping QEMU through the virtqueue family"
order: 1
description: "virtio is QEMU's paravirtual device framework: the guest driver and QEMU share virtqueues, ring structures in guest memory holding descriptors that point at guest buffers. QEMU walks these descriptor chains to perform I/O, so flaws in descriptor handling, chain length, indirect descriptors, or the virtio configuration, give out-of-bounds access in the QEMU process."
keywords:
  - virtio
  - virtqueue
  - vring
  - descriptor
  - indirect descriptor
---

# virtio devices

virtio is the paravirtual I/O framework shared by virtio-net, virtio-blk, virtio-scsi, virtio-gpu, and others. The guest driver and QEMU communicate through virtqueues: each is a vring with a descriptor table, an available ring, and a used ring, all in guest memory. A descriptor holds a guest physical address, a length, and flags; descriptors chain, and an indirect descriptor points at a further table of descriptors. QEMU reads these guest-controlled structures to locate buffers for each operation, so the handling of descriptor chains, lengths, indirect tables, and the device configuration space is the escape surface, and it is broad because every virtio device shares it.

## The virtqueue surface

```c
// a vring descriptor (guest-controlled):
struct vring_desc { uint64_t addr; uint32_t len; uint16_t flags; uint16_t next; };
// QEMU maps addr for len bytes and follows next when VRING_DESC_F_NEXT is set,
// or treats the buffer as a descriptor table when VRING_DESC_F_INDIRECT is set.
// Primitives a malicious guest driver builds:
//  - a descriptor length QEMU uses for a copy that exceeds the mapped region
//  - a chain or indirect table forming a loop or huge count -> over-read/-write
//  - an available/used ring index advanced past the queue size
//  - per-device config/header fields (e.g. virtio-net header, virtio-gpu commands)
//    with lengths or counts QEMU trusts
```

Because the guest writes the descriptor table and ring indices directly, it controls the addresses, lengths, and linkage QEMU follows; the recurring bugs are a trusted length used for a memory operation and an index or count not bounded against the queue or buffer.

## Per-device specifics

```bash
# virtio-net: the virtio-net header and packet buffers; offload fields
# virtio-blk/scsi: request headers with sector/length fields
# virtio-gpu: a command stream (resource create, transfer, present) with guest sizes
lspci -nn | grep -i virtio; dmesg | grep -i virtio
```

virtio-gpu is notable for parsing a command stream with resource and transfer descriptors, similar in spirit to the VMware SVGA 3D surface; virtio-net and virtio-blk expose header and request parsing.

## Exploitation notes

- The shared virtqueue layer means a descriptor-handling bug can be reached through whichever virtio device is present; indirect descriptors are a classic hot spot because they add a second guest-controlled table QEMU must bound.
- virtio-gpu widens the surface with a command/resource model on top of the virtqueue, a richer parse target akin to other hypervisors' 3D paths.
- The guest drives this from a controlled virtio driver that writes raw descriptors and rings, not through the normal kernel stack, giving precise control of the structures QEMU reads.
- Primitives land in the QEMU process heap; a full escape adds a leak and heap groom, and is QEMU-version-specific.

## References

- [virtio specification (OASIS)](https://docs.oasis-open.org/virtio/virtio/v1.2/virtio-v1.2.html)
- [QEMU security advisories](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM escape: virtio](https://github.com/WinMin/Awesome-VM-Exploit)
