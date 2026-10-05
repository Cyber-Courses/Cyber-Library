---
title: "Firecracker: escaping the AWS microVM monitor"
description: "Attacking AWS Firecracker: escaping the microVM through its minimal virtio device models (net, block, vsock), and abusing the jailer and the control API socket that configure and launch each microVM, within the deliberately small surface Firecracker exposes."
keywords:
  - Firecracker
  - microVM
  - virtio
  - jailer
  - VM escape
---

# Firecracker

Firecracker is the minimal VMM behind AWS Lambda and Fargate and many sandbox runtimes. It deliberately emulates only a few virtio devices (net, block, vsock) and a serial console, with no BIOS, PCI, or legacy devices, which is a much smaller escape surface than QEMU. What remains is those virtio device models, reached from the guest, and the control API socket and the jailer that launch and confine each microVM, reached from the host side.

```text
Firecracker attack surface:
- virtio device models: net, block, vsock (from the guest)
- The REST control API socket (host side, per microVM)
- The jailer (seccomp, cgroups, chroot confinement)
```

## Exploitation notes

- The guest-to-host surface is small by design, so escapes target the virtio device implementations; a bug there lands in the Firecracker process.
- The jailer wraps Firecracker in seccomp, cgroups, and a chroot, so an escape is bounded by that confinement, enumerate it before assuming host access.
- Firecracker backs sandboxed container runtimes; the container-side view is [Kata Containers](../../../containers/container-escape/sandboxed-runtime-escapes/kata-containers.md).

## References

- [Firecracker design](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md)
- [Firecracker security](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md#security)
