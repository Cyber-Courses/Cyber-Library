---
title: "Network adapters: escaping VirtualBox through e1000 and PCNet"
description: "Escaping a VirtualBox guest through its emulated network adapters, the Intel e1000 and AMD PCNet models, a recurring and productive escape surface whose descriptor and packet handling runs in the host-side VM process."
keywords:
  - e1000
  - PCNet
  - network adapter
  - VirtualBox escape
  - descriptor ring
---

# Network adapters

VirtualBox's emulated NICs, the Intel e1000 and the AMD PCNet, process the guest's transmit and receive descriptor rings and packet buffers in the host-side VM process. These adapters have a well-documented history of guest-to-host escapes, with crafted descriptors and packet data triggering memory corruption on the host.

```text
Network-adapter escape surface:
- e1000: TX/RX descriptor rings, offload handling
- PCNet: descriptor and buffer handling
```

## Exploitation notes

- The e1000 is present by default on many guest types, making it broadly reachable.
- Public, complete guest-to-host exploit chains exist for these adapters, a good source of concrete technique.
- Code execution lands in the host VM process as the launching user, then escalates.

## References

- [Oracle VirtualBox manual](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox research](https://www.zerodayinitiative.com/blog)
