---
title: "Firecracker: attacking the AWS microVM monitor"
order: 1
description: "Firecracker emulates only virtio-block, virtio-net, virtio-vsock, a serial console, and a minimal controller set, and is written in Rust with a seccomp jailer. The guest-to-host surface is those virtio devices and their virtqueue handling; the host side adds the REST control API and the jailer configuration, where misconfiguration or a logic flaw weakens the intended isolation."
keywords:
  - firecracker
  - virtio
  - jailer
  - seccomp
  - rest api
---

# Firecracker

Firecracker runs each microVM with a tiny device set: virtio-block, virtio-net, virtio-vsock, a serial console, a minimal interrupt controller, and a few platform devices, nothing else. It is written in Rust and shipped with a `jailer` that wraps each instance in a seccomp filter, a chroot, and dedicated namespaces and cgroups. The guest-to-host escape surface is therefore narrow: the virtio devices and their virtqueue handling. The host-side surface is the REST control API (on a Unix socket) and the jailer's confinement, where a weak configuration or a logic flaw in device or API handling is the realistic target rather than a classic memory-corruption bug.

## The surfaces

```bash
# guest side: the only paravirtual devices present
lspci 2>/dev/null; ls /sys/bus/virtio/devices 2>/dev/null   # virtio-mmio block/net/vsock
# host side: the control API socket (per microVM)
curl --unix-socket /run/firecracker.socket -s http://localhost/machine-config
```

```text
Escape and abuse surface:
- virtio-block / -net / -vsock virtqueue descriptor handling (the only device parsing)
- the vsock device, which bridges guest and host sockets, is a notable logic surface
- the REST API: whoever reaches the control socket can reconfigure/boot/stop the VM,
  attach drives (host files), and set up networking - control of the microVM
- the jailer: if the seccomp profile, chroot, or namespaces are weakened or bypassed,
  a device-layer bug that would be contained becomes a broader host compromise
```

## Exploitation notes

- The virtio virtqueue handling is the guest-to-host surface; the mechanisms match [virtio devices](../qemu/guest-to-host-escape/virtio-devices.md), but the implementation is Rust, so the realistic bugs are logic errors and `unsafe` blocks rather than ubiquitous overflows.
- vsock is a distinctive surface because it bridges guest sockets to host sockets; its connection and buffer handling is a logic-flaw target.
- The REST API socket is effectively microVM control: reaching it lets an attacker attach host files as drives or reconfigure the VM, so its access control (socket permissions, the jailer) is critical.
- The jailer's seccomp, chroot, and namespace confinement is the defense-in-depth layer; its strength determines whether a device-layer compromise stays contained, so assess the jailer configuration.

## References

- [Firecracker design](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md)
- [Firecracker jailer](https://github.com/firecracker-microvm/firecracker/blob/main/docs/jailer.md)
- [Firecracker API](https://github.com/firecracker-microvm/firecracker/blob/main/src/firecracker/swagger/firecracker.yaml)
