---
title: "Synthetic devices: escaping Hyper-V through the worker process"
description: "Escaping Hyper-V through the synthetic storage, video, and input devices serviced by the VM worker process (vmwp.exe) in the parent partition, which parses guest-controlled device data and, when flawed, yields code execution in the worker process."
keywords:
  - synthetic device
  - vmwp.exe
  - worker process
  - Hyper-V escape
  - VSP
---

# Synthetic devices

Each Hyper-V VM has a worker process, `vmwp.exe`, in the parent partition that services the guest's synthetic devices (storage, video, input, and others) through the virtualization service providers. The worker process parses the device requests and data the guest sends over VMBus, so a memory-corruption flaw runs code in `vmwp.exe`, which is then escalated to SYSTEM on the host.

```text
Synthetic-device escape surface:
- Synthetic storage, video, and input request handling in vmwp.exe
- The virtualization service provider (VSP) interfaces
```

## Exploitation notes

- The worker process runs per VM with reduced privileges, so a bug yields worker-process code that is chained to SYSTEM; it is a step less direct than a [Virtual switch](virtual-switch.md) kernel bug.
- Which devices are reachable depends on the guest's configured synthetic hardware.
- The transport beneath these devices is [VMBus](vmbus.md).

## References

- [Microsoft Security Research: attacking the VM worker process](https://microsoft.github.io/Attacking-the-VM-Worker-Process/)
- [Microsoft Hyper-V bug bounty](https://www.microsoft.com/en-us/msrc/bounty-hyper-v)
