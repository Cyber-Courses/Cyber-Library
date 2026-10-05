---
title: "Network adapters: escaping VirtualBox through emulated NICs"
description: "VirtualBox emulates the Intel e1000, the AMD PCnet, and virtio-net adapters, whose guest drivers program descriptor rings the host VM process reads to move packets. Flaws in descriptor handling, in offload and loopback processing, and in the PCnet and e1000 transmit paths give out-of-bounds access in the host process, a recurring VirtualBox escape surface."
keywords:
  - e1000
  - pcnet
  - virtio-net
  - descriptor
  - offload
---

# Network adapters

VirtualBox emulates several NICs: the Intel e1000 (and e1000e), the AMD PCnet (Am79C97x), and virtio-net. Each has transmit and receive descriptor rings the guest programs in its memory, and the host VM process reads those descriptors and the packet buffers to move traffic. The transmit path and offload handling parse guest-chosen lengths and metadata, so the NIC models have repeatedly produced out-of-bounds reads and writes in VirtualBox, with the PCnet and e1000 transmit descriptor handling being notable loci.

## The surface

```c
// e1000: TX descriptor processing and segmentation/checksum offload; a TX length or
//   offload combination the model copies without bounding -> OOB read into the host
// PCnet: the legacy Am79C97x transmit/receive descriptor rings; crafted descriptors
//   (lengths, chaining) have driven out-of-bounds access in the emulation
// virtio-net: the virtio-net header and virtqueue descriptors (shared virtio surface)
```

```bash
lspci -nn | grep -iE 'ethernet|intel|amd'   # which NIC model is configured
# an attacker guest NIC driver programs the rings and descriptors directly
```

## Exploitation notes

- Transmit-side processing and offload are the usual bug loci: the host computes over guest-chosen lengths and header fields, and loopback modes feed crafted frames back through host parsing.
- The PCnet model is a legacy device still offered and has its own descriptor-handling history; the e1000 model is shared in spirit with other hypervisors, giving cross-product relevance.
- The guest controls the rings and descriptors from its driver, so exploitation programs the device directly rather than sending normal packets; virtio-net routes through the shared virtqueue layer.
- Primitives land in the VM process; version-specific, pair with a leak.

## References

- [VirtualBox manual: networking](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox network research](https://www.zerodayinitiative.com/blog)
- [Oracle security alerts](https://www.oracle.com/security-alerts/)
