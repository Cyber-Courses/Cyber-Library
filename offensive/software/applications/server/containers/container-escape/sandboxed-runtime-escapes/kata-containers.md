---
title: "Kata Containers: escaping the microVM guest to the host"
description: "Escaping Kata Containers and similar microVM runtimes, where each workload runs in a lightweight virtual machine, by breaking out of the guest VM through a hypervisor device or a shared filesystem interface rather than through Linux container isolation."
keywords:
  - Kata Containers
  - microVM
  - hypervisor escape
  - virtio
  - sandbox escape
---

# Kata Containers

Kata runs each pod or container inside a lightweight virtual machine, so the isolation boundary is the hypervisor, not namespaces. A container escape in the Linux sense only reaches the guest kernel; reaching the host requires a VM escape through the hypervisor's emulated devices (virtio, QEMU or Firecracker device models) or the shared-filesystem channel (virtio-fs) used to pass files in.

```bash
# Confirm the microVM: the guest sees virtio devices and a Kata-provided kernel
lspci 2>/dev/null | grep -i virtio
cat /proc/cmdline            # Kata/agent-specific boot parameters
```

## Exploitation notes

- Escaping the guest kernel is not enough; the exploit must then break the hypervisor, so this reduces to a VM-escape problem against the configured monitor.
- The shared-filesystem path (virtio-fs) and any passed-through devices are the highest-value host interfaces to audit.
- As with gVisor, fingerprint the runtime first, since microVM isolation invalidates ordinary container-escape primitives.

## References

- [Kata Containers architecture](https://katacontainers.io/learn/)
- [Firecracker design](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md)
