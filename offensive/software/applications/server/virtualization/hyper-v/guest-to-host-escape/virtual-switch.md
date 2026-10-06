---
title: "Virtual switch: escaping Hyper-V through the networking datapath"
order: 1
description: "The Hyper-V virtual switch forwards VM network traffic in the root partition, with an extensible filter stack and the synthetic and emulated NIC paths feeding it. Guest-controlled packets and the synthetic network VSP requests are parsed there, so flaws in the switch, its extensions, or the network VSP give memory corruption in the root-partition networking stack."
keywords:
  - hyper-v virtual switch
  - vmswitch
  - netvsp
  - packet parsing
  - root partition
---

# Virtual switch

The Hyper-V virtual switch (`vmswitch`) forwards network traffic for every VM, running in the root partition kernel. Guest traffic reaches it through the synthetic network VSP (`netvsp`) over VMBus or through an emulated NIC, and the switch applies an extensible filter stack before forwarding. Because the switch and the network VSP parse guest-controlled packets and request structures in the privileged root partition, flaws there, in packet parsing, offload handling, or the VSP's ring and descriptor processing, corrupt the root-partition networking stack and have been a notable Hyper-V escape surface.

## The surface

```c
// netvsp receives send/receive buffer descriptors and RNDIS control/data messages
// from the guest over VMBus. Primitives:
//  - an RNDIS message with a length or offset field vmswitch/netvsp trusts
//  - send/receive buffer section descriptors with out-of-range offsets or counts
//  - offload (checksum/segmentation) metadata the switch parses from guest packets
// a trusted length used for a copy, or an index not bounded, -> OOB in the root partition
```

RNDIS is the control protocol the synthetic NIC uses; its messages (set/query OIDs, packet descriptors) are parsed host-side, and malformed RNDIS has been a recurring source of `vmswitch` bugs.

## Exploitation notes

- The synthetic NIC path (netvsp plus vmswitch) is the richer target than the emulated NIC, because it parses the RNDIS protocol and the send/receive buffer descriptor model in the root-partition kernel.
- `vmswitch` runs in the root partition kernel, so corruption is a kernel-level primitive on the host, among the highest-impact Hyper-V escapes.
- The guest drives this by sending crafted RNDIS messages and buffer descriptors from its NIC driver; an attacker guest driver controls them directly.
- The transport is [VMBus](vmbus.md); the network VSP is one of the [synthetic devices](synthetic-devices.md), and this page is the switch and RNDIS specifics.

## References

- [Microsoft: Hyper-V virtual switch](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v-virtual-switch/hyper-v-virtual-switch)
- [MSRC: vmswitch research](https://www.microsoft.com/en-us/msrc)
