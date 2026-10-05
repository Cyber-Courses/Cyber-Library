---
title: "Virtual switch: escaping Hyper-V through vmswitch"
description: "Escaping Hyper-V through the virtual switch (vmswitch), which parses guest network frames in the host kernel, the most dangerous Hyper-V escape surface because it is reachable from any guest with a virtual NIC and a flaw there yields host kernel code execution."
keywords:
  - vmswitch
  - virtual switch
  - host kernel
  - Hyper-V escape
  - network
---

# Virtual switch

The Hyper-V virtual switch, `vmswitch`, runs in the host kernel and processes the network frames guests send. Because it parses guest-controlled packet data in the most privileged context, a memory-corruption flaw there is host kernel code execution, reached from any guest that has a virtual network adapter. This makes it the highest-impact Hyper-V escape surface.

```text
vmswitch escape surface:
- Guest frame parsing and header handling in the host kernel
- Offload and extension processing
```

## Exploitation notes

- It is reachable from any guest with a virtual NIC, with no special configuration, which is what makes it so dangerous.
- Success lands directly in the host kernel, bypassing the worker-process boundary that bounds synthetic-device bugs.
- This is a recurring high-severity Hyper-V class in Microsoft's advisories.

## References

- [Microsoft Hyper-V bug bounty](https://www.microsoft.com/en-us/msrc/bounty-hyper-v)
- [Microsoft: Hyper-V architecture](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
