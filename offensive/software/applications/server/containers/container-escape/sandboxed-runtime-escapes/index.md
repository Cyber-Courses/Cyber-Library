---
title: "Sandboxed runtime escapes: breaking out of gVisor and Kata isolation"
description: "Sandboxed runtimes add a layer between the container and the host kernel: gVisor interposes a userspace kernel, and Kata Containers runs each container in a lightweight virtual machine. Escaping them means defeating that added layer, through a bug in the userspace kernel's syscall emulation and file proxy, or a VM escape and the Kata agent, before the usual host compromise."
keywords:
  - gvisor
  - kata containers
  - sandbox escape
  - sandboxed runtime
  - container escape
---

# Sandboxed runtime escapes

Standard containers share the host kernel directly, so a kernel bug is a host bug. Sandboxed runtimes insert a barrier. gVisor runs a userspace kernel (the Sentry) that intercepts the container's syscalls so they never reach the host kernel directly, with a separate file proxy (the Gofer) mediating filesystem access. Kata Containers runs each container inside a lightweight virtual machine with its own guest kernel, so the container is isolated from the host by the hypervisor. Escaping either requires defeating the added layer first, and only then do the familiar host techniques apply.

Detect which runtime is in use:

```bash
dmesg 2>/dev/null | grep -i gvisor
cat /proc/version 2>/dev/null                 # gVisor reports a distinctive version string
mount | grep -i kata; ls /dev | grep -i kata  # Kata guest artefacts
uname -a                                       # a minimal guest kernel suggests a VM-based sandbox
```

## Subtopics

- **[gVisor](gvisor.md)**: escaping the userspace kernel and its file proxy.
- **[Kata Containers](kata-containers.md)**: escaping the guest VM and reaching the host through the agent.

## References

- [gVisor security model](https://gvisor.dev/docs/architecture_guide/security/)
- [Kata Containers architecture](https://github.com/kata-containers/kata-containers/blob/main/docs/design/architecture/README.md)
