---
title: "Network adapters: escaping QEMU through emulated NICs"
order: 2
description: "QEMU emulates several NICs, the Realtek rtl8139, Intel e1000/e1000e, and virtio-net, whose transmit and receive paths parse guest-programmed descriptors and packet data. Flaws in descriptor handling, in offload and loopback processing, and in packet reassembly give out-of-bounds access in the QEMU process from the guest network stack."
keywords:
  - e1000
  - rtl8139
  - virtio-net
  - descriptor
  - offload
---

# Network adapters

A QEMU guest's NIC is emulated, commonly the Realtek rtl8139, Intel e1000/e1000e, or paravirtual virtio-net. Each has transmit and receive descriptor rings the guest programs in its memory, and QEMU reads those descriptors and the packet buffers they reference to move traffic. The transmit path in particular parses guest-chosen lengths and offload metadata, and features like loopback and checksum/segmentation offload add host-side processing of guest data, so NIC models have repeatedly produced out-of-bounds and infinite-loop bugs in QEMU.

## The surface by model

```c
// rtl8139: the transmit path and C+ mode descriptors; offload/loopback handling
//   has produced out-of-bounds reads where a length is trusted during TX processing
// e1000/e1000e: TX descriptor processing, segmentation/checksum offload, and
//   descriptor-ring handling; a TX descriptor length/offload combination used for a
//   copy, or a ring that the model walks without bounding, -> OOB in QEMU
// virtio-net: the virtio-net header and virtqueue descriptors (see virtio devices)
```

```bash
lspci -nn | grep -iE 'ethernet|realtek|intel'   # which NIC model
# an attacker guest NIC driver programs the rings and writes descriptors directly,
# controlling lengths, buffer pointers, and offload fields
```

The recurring patterns are a transmit length or offload field the model uses for a copy that exceeds the source buffer, and ring/descriptor processing that loops or reads past bounds on crafted input.

## Exploitation notes

- Transmit-side offload (TCP segmentation, checksum) is a frequent bug locus because the model computes over guest-chosen header fields and lengths; loopback modes feed crafted frames straight back through host parsing.
- The e1000 model is shared with other hypervisors (including VMware), so e1000 bugs often have cross-product relevance; virtio-net routes through the shared virtqueue layer, see [virtio devices](virtio-devices.md).
- The guest controls the rings and descriptors from its driver, so exploitation programs the device directly rather than sending ordinary packets.
- Primitives land in the QEMU process; pair with a leak and groom, QEMU-version-specific.

## References

- [QEMU network device documentation](https://www.qemu.org/docs/master/system/devices/net.html)
- [QEMU security advisories](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM escape: network](https://github.com/WinMin/Awesome-VM-Exploit)
