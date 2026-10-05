---
title: "Guest to host escape: breaking out of a Hyper-V child partition"
description: "Escaping a Hyper-V guest to the host by exploiting the synthetic device stack that bridges child and parent partitions: the virtual switch in the host kernel, the synthetic devices serviced by the VM worker process, and the VMBus channel transport underneath them."
keywords:
  - Hyper-V escape
  - vmswitch
  - vmwp.exe
  - VMBus
  - guest to host
---

# Guest to host escape

A Hyper-V child partition talks to the host through synthetic devices over VMBus. The virtual switch parses guest network frames in the host kernel, the VM worker process (`vmwp.exe`) services synthetic storage, video, and input devices in the parent partition, and VMBus carries all of it. Each parses guest-controlled data, so flaws there run code in the host, in the kernel or the worker process depending on the component.

## Subtopics

- **[Virtual switch](virtual-switch.md)**: guest network frames parsed in the host kernel.
- **[Synthetic devices](synthetic-devices.md)**: storage, video, and input in the worker process.
- **[VMBus](vmbus.md)**: the channel and ring-buffer transport.

## References

- [Microsoft Security Research: attacking the VM worker process](https://microsoft.github.io/Attacking-the-VM-Worker-Process/)
- [Microsoft Hyper-V bug bounty](https://www.microsoft.com/en-us/msrc/bounty-hyper-v)
