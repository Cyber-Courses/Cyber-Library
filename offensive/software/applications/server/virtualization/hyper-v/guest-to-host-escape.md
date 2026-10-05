---
title: "Guest to host escape: breaking out of a Hyper-V child partition"
description: "Escaping a Hyper-V guest to the host by exploiting the synthetic device stack that bridges child and parent partitions: VMBus channels, the virtual switch, and the VM worker process vmwp.exe that runs in the host and parses guest-controlled data."
keywords:
  - Hyper-V escape
  - VMBus
  - vmwp.exe
  - virtual switch
  - guest to host
---

# Guest to host escape

A Hyper-V child partition talks to the host through synthetic devices over VMBus, serviced by the `vmwp.exe` worker process (one per VM) in the parent partition and by kernel components. Those components parse guest-controlled data, so memory-corruption flaws in VMBus device handling, the virtual switch, or the worker process let a guest run code in the host context.

```text
Attack surface reachable from a guest:
- VMBus channel messages to synthetic devices (storage, network, video, input)
- The virtual switch (vmswitch) parsing guest network frames
- vmwp.exe handling of device and configuration data in the parent partition
```

## Exploitation notes

- The worker process runs per VM in the parent partition with reduced privileges, so a `vmwp.exe` bug yields host user code that is then chained to SYSTEM; kernel VMBus bugs yield host kernel directly.
- The highest-value surface is the synthetic device and vmswitch code, which is reachable from an unprivileged guest without any host credential.
- Named, weaponized instances are collected under [Known escape exploits](known-escape-exploits.md).

## References

- [Microsoft: Hyper-V architecture](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [Microsoft Security Research: attacking the Hyper-V stack](https://microsoft.github.io/Attacking-the-VM-Worker-Process/)
