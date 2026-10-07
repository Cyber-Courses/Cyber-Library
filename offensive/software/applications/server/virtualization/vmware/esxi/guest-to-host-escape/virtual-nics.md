---
title: "Virtual NICs: escaping ESXi through the e1000 and vmxnet3 adapters"
order: 3
description: "ESXi offers the emulated e1000 and the paravirtual vmxnet3 network adapters. The guest driver programs transmit and receive descriptor rings in guest memory that the vmx process reads via DMA to move packets. Flaws in descriptor and ring handling, including the vmxnet3 command interface, give out-of-bounds access in the host process from the guest network stack."
keywords:
  - vmxnet3
  - e1000
  - descriptor ring
  - dma
  - network adapter
---

# Virtual NICs

A guest's network adapter is emulated: the legacy Intel e1000, or VMware's paravirtual vmxnet3. In both, the guest driver sets up transmit and receive descriptor rings in guest memory and programs the device with their base addresses and lengths through MMIO or I/O registers; the `vmx` process then reads descriptors by DMA to send and receive packets. Walking those guest-controlled rings and descriptors in the host process is the escape surface, and vmxnet3's additional command and configuration interface adds structures the host parses from guest memory.

## Driving the adapter

```bash
lspci -nn | grep -iE 'ethernet|network'    # e1000 or vmxnet3
dmesg | grep -iE 'e1000|vmxnet3'
# an attacker guest driver programs the ring base/length registers and writes
# descriptors directly, controlling lengths, flags, and buffer pointers
```

For e1000 the transmit path reads TX descriptors whose length and buffer address the guest controls; classic flaws come from how the model handles descriptor-specified lengths and segmentation offload, where a length used for a copy exceeds the backing buffer. For vmxnet3 the guest configures the device through a shared memory region and a command register, defining the TX/RX queues, their ring sizes, and offload parameters; the host parses this configuration and the per-descriptor fields, so an inconsistent ring length, an out-of-range descriptor index, or a crafted offload field drives the host out of bounds.

```c
// e1000: a TX descriptor with a length/offload combination the model copies
// without bounding against the mapped buffer -> OOB read into the host
// vmxnet3: a device configuration (via the command register + shared area) with
// a ring length or descriptor index the host trusts -> OOB access when it walks
```

## Exploitation notes

- The host trusts the ring base, lengths, and descriptor fields the guest programs; the reliable primitives are a transmit length that overruns the source buffer and a receive ring index or size the host uses without bounds checking.
- vmxnet3's configuration interface is the richer target because the host parses a guest-defined device setup (queues, rings, offloads) in addition to per-packet descriptors, widening the parse surface.
- Packet-content parsing (checksum and segmentation offload) adds host-side processing of guest-chosen header fields, another place length assumptions break.
- A working escape combines the OOB primitive with a leak and is build-specific; the e1000 model is shared with other hypervisors, so e1000 bugs often have cross-product relevance.

## References

- [Intel e1000 software developer's manual](https://www.intel.com/content/www/us/en/support/products/1285/ethernet-products.html)
- [vmxnet3 driver (open-vm-tools)](https://github.com/vmware/open-vm-tools)
- [Zero Day Initiative: VMware network device research](https://www.zerodayinitiative.com/blog)
