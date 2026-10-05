---
title: "Cloud Hypervisor: attacking the Rust VMM"
description: "Cloud Hypervisor is a Rust KVM monitor with a modern but still limited device set, virtio devices, optional PCI and device passthrough, and an HTTP control API. The guest-to-host surface is its virtio and VFIO handling and the shared rust-vmm components; the host surface is the API and the configuration of passthrough and sandboxing."
keywords:
  - cloud hypervisor
  - rust-vmm
  - virtio
  - vfio
  - http api
---

# Cloud Hypervisor

Cloud Hypervisor is a Rust-based KVM monitor aimed at modern cloud and container workloads. It offers more than Firecracker, virtio devices, optional PCI, and VFIO device passthrough, while still omitting most legacy emulation, and shares virtio and VMM building blocks with the `rust-vmm` project. The guest-to-host escape surface is its virtio device handling, the VFIO passthrough path when enabled, and the shared rust-vmm components; the host-side surface is the HTTP control API and the configuration choices (passthrough, sandboxing) that widen or narrow exposure.

## The surfaces

```bash
# guest: the virtio (and optionally passthrough PCI) devices
ls /sys/bus/virtio/devices 2>/dev/null; lspci 2>/dev/null
# host: the HTTP API socket controls the VM lifecycle and device config
curl --unix-socket /run/cloud-hypervisor.sock -s http://localhost/api/v1/vm.info
```

```text
Escape and abuse surface:
- virtio device virtqueue handling (shared rust-vmm/virtio code) - the main guest surface
- VFIO device passthrough, when configured: a passed-through physical device widens
  the surface to that device's driver and DMA, and weakens isolation versus emulation
- the HTTP API: lifecycle and device (disk/net) configuration, i.e. VM control
- shared rust-vmm components mean a bug there can affect multiple VMMs built on them
```

## Exploitation notes

- The virtio virtqueue handling is the guest-to-host surface, implemented in Rust via shared rust-vmm code, so logic flaws and `unsafe` usage are the realistic bug classes; see [virtio devices](../qemu/guest-to-host-escape/virtio-devices.md) for the mechanism.
- VFIO passthrough, when enabled, is a double-edged configuration: it gives a VM direct access to a physical device, enlarging the surface to that device and its DMA and reducing the isolation that pure emulation provides.
- The HTTP API is VM control: reaching its socket allows reconfiguring devices and the VM lifecycle, so its access control matters as much as the device code.
- Shared rust-vmm components create cross-VMM relevance: a flaw in a shared virtio crate can apply to other monitors built on rust-vmm.

## References

- [Cloud Hypervisor documentation](https://www.cloudhypervisor.org/docs/)
- [rust-vmm project](https://github.com/rust-vmm)
- [Cloud Hypervisor API](https://github.com/cloud-hypervisor/cloud-hypervisor/blob/main/docs/api.md)
