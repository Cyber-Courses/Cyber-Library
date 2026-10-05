---
title: "Guest-to-host escape: breaking out of a Hyper-V virtual machine"
description: "A Hyper-V guest escapes by corrupting the root partition code that services its requests: the VMBus channel transport and the synthetic devices (storage, network, video, HID) backed by Virtualization Service Providers, and the emulated devices and virtual switch in the worker process. A memory-safety flaw there executes in the root partition, which controls every VM."
keywords:
  - hyper-v escape
  - vmbus
  - vsp
  - synthetic devices
  - root partition
---

# Guest-to-host escape

A Hyper-V guest is isolated by the hypervisor, so it cannot touch the host directly; it reaches host functionality through VMBus, over which synthetic devices talk to Virtualization Service Providers (VSPs) in the root partition, and through the worker process (`vmwp.exe`) that handles emulated devices and the virtual switch. Escaping means making one of those root-partition components mishandle guest-controlled data: a VMBus packet, a synthetic-device request, or an emulated-device access. Code execution lands in the root partition or the worker process, which is effectively host control because the root partition administers all guests.

```powershell
# the VMBus devices and channels visible from the guest
Get-PnpDevice | Where-Object InstanceId -like 'VMBUS*'
# synthetic storage/net/video/HID are the high-level targets; the transport is VMBus
```

## Subtopics

- **[VMBus](vmbus.md)**: the ring-buffer channel transport between guest and root partition.
- **[Synthetic devices](synthetic-devices.md)**: the VSP-backed storage, network, video, and HID devices.
- **[Virtual switch](virtual-switch.md)**: the networking datapath in the root partition.

## References

- [Microsoft: Hyper-V architecture and VMBus](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [MSRC: Hyper-V security research](https://www.microsoft.com/en-us/msrc)
