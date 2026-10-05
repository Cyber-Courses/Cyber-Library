---
title: "Network adapters: escaping through the QEMU e1000 and rtl8139 models"
description: "Escaping a KVM guest through the QEMU emulated network adapters, the Intel e1000 and Realtek rtl8139 models, whose descriptor-ring processing and offload handling parse guest-controlled packet data in the host QEMU process."
keywords:
  - e1000
  - rtl8139
  - network adapter
  - descriptor ring
  - QEMU escape
---

# Network adapters

QEMU's emulated NICs process the guest's transmit and receive descriptor rings and packet buffers in the host process. The Intel e1000 and Realtek rtl8139 models are the classic targets: e1000 for its descriptor and segmentation-offload handling, rtl8139 for its C+ mode and loopback path. Crafted descriptors or packets trigger memory corruption in the host QEMU process.

```text
Network-adapter escape surface:
- e1000: TX/RX descriptor rings, TCP segmentation offload
- rtl8139: C+ mode descriptors, the loopback transmit path
```

## Exploitation notes

- These legacy adapters are commonly chosen for compatibility, so they are frequently present on guests; the machine type and NIC model determine reachability.
- The rtl8139 loopback path is a well-known information-leak and corruption surface reachable without network connectivity.
- virtio-net is the modern alternative surface; see [virtio devices](virtio-devices.md).

## References

- [QEMU network emulation](https://www.qemu.org/docs/master/system/devices/net.html)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
