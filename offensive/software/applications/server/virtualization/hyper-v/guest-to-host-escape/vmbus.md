---
title: "VMBus: escaping Hyper-V through the channel transport"
description: "Escaping Hyper-V through flaws in VMBus, the channel and ring-buffer transport that carries all synthetic-device communication between a guest and the parent partition, where ring and packet handling errors corrupt host state before any device logic runs."
keywords:
  - VMBus
  - ring buffer
  - channel
  - Hyper-V escape
  - transport
---

# VMBus

VMBus is the transport underneath every synthetic device: a set of channels backed by shared-memory ring buffers that carry packets between the guest and the parent partition. The host-side VMBus code (in the kernel and the worker process) parses channel setup, ring indices, and packet headers. Flaws there corrupt host state before any higher-level device handler runs, so VMBus is a transport-level escape surface shared by all synthetic devices.

```text
VMBus escape surface:
- Ring-buffer index and packet-header handling
- Channel offer, open, and GPADL (memory-region) setup
```

## Exploitation notes

- Because VMBus underlies all synthetic devices, a transport bug is broadly reachable from any guest that uses them.
- Depending on where the flaw is, it lands in the host kernel or the worker process; see [Synthetic devices](synthetic-devices.md).
- Ring-buffer index handling is a classic corruption point in shared-ring transports.

## References

- [Microsoft Security Research: attacking the VM worker process](https://microsoft.github.io/Attacking-the-VM-Worker-Process/)
- [Microsoft: Hyper-V architecture](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
