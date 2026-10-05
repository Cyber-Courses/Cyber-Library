---
title: "Cloud Hypervisor: escaping the Rust virtual machine monitor"
description: "Attacking Cloud Hypervisor, the Rust virtual machine monitor for modern cloud workloads: escaping the microVM through its virtio device models, and abusing its control API socket, within a surface kept small and written in a memory-safe language."
keywords:
  - Cloud Hypervisor
  - Rust VMM
  - virtio
  - microVM
  - VM escape
---

# Cloud Hypervisor

Cloud Hypervisor is a modern VMM written in Rust, used for cloud and sandboxed workloads and sharing the rust-vmm components with other projects. Like Firecracker it exposes a minimal, mostly virtio device set and a control API socket. Being written in a memory-safe language removes many classic memory-corruption bugs, so the surface is its virtio device logic, any unsafe or FFI code, and the control API.

```text
Cloud Hypervisor attack surface:
- virtio device models (net, block, fs, vsock) from the guest
- unsafe / FFI boundaries in the device and memory code
- The REST control API socket (host side)
```

## Exploitation notes

- Memory safety raises the bar on traditional overflow bugs, pushing attention to logic flaws, `unsafe` blocks, and shared rust-vmm components.
- An escape lands in the Cloud Hypervisor process, bounded by whatever seccomp and user confinement the deployment applies.
- Like Firecracker, it backs microVM-isolated containers; the container-side view is [Kata Containers](../../../containers/container-escape/sandboxed-runtime-escapes/kata-containers.md).

## References

- [Cloud Hypervisor](https://www.cloudhypervisor.org/)
- [rust-vmm project](https://github.com/rust-vmm)
