---
title: "Virtual NICs: escaping ESXi through vmxnet3 and e1000"
description: "Escaping ESXi through the virtual network adapters, the paravirtualized vmxnet3 and the emulated e1000, whose descriptor-ring and offload handling in the vmx process has produced host code execution from a guest."
keywords:
  - vmxnet3
  - e1000
  - virtual NIC
  - vmx
  - ESXi escape
---

# Virtual NICs

ESXi's virtual network adapters process the guest's transmit and receive descriptor rings in the `vmx` process. The paravirtualized vmxnet3 adapter is the common default and a notable escape surface, and the emulated e1000 is also present. Crafted descriptors or offload parameters trigger memory corruption in the host-side process.

```text
Virtual-NIC escape surface:
- vmxnet3: TX/RX queues, descriptor and offload handling
- e1000: descriptor rings and segmentation offload
```

## Exploitation notes

- vmxnet3 is the default high-performance adapter, so it is widely present; its descriptor handling is the reachable surface.
- The adapter processing runs in the `vmx` process, then escalates to the host.
- The same adapter code is shared with Workstation and Fusion.

## References

- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
