---
title: "Container escape: breaking out of a container to the host"
description: "Runtime-agnostic container breakout: escaping to the host through over-permissive configuration (the privileged flag, dangerous capabilities, device access, weak confinement), dangerous bind mounts, shared host namespaces, runtime and kernel exploits, and escapes from sandboxed runtimes."
keywords:
  - container escape
  - container breakout
  - privileged container
  - host namespace
  - container runtime exploit
---

# Container escape

A container escape turns code execution inside a container into code execution on the host. It almost never depends on the engine: the same primitives work from Docker, Podman, containerd, and a Kubernetes pod, because they all lean on the same kernel features. What differs is only how the dangerous setting was handed to the container, which is why the runtime and orchestration layers link here rather than re-document the breakout.

The primitives group by what makes the escape possible.

## Subtopics

- **[Privileged configuration](privileged-configuration/index.md)**: the container was given too much, from the all-in-one privileged flag to a single dangerous capability, raw device access, or a disabled seccomp or AppArmor profile.
- **[Sensitive mounts](sensitive-mounts/index.md)**: a host path, a runtime control socket, or host procfs and sysfs was bind-mounted into the container.
- **[Shared host namespaces](shared-host-namespaces/index.md)**: the container shares the host PID, network, IPC, or user namespace.
- **[Runtime and kernel exploits](runtime-and-kernel-exploits/index.md)**: a code-execution flaw in the OCI runtime or shim, or in the shared host kernel.
- **[Sandboxed runtime escapes](sandboxed-runtime-escapes/index.md)**: breaking out of a sandbox that interposes on the kernel, such as gVisor or a microVM runtime.

## References

- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [NCC Group: Understanding and Hardening Linux Containers](https://research.nccgroup.com/2016/04/13/understanding-and-hardening-linux-containers/)
- [man 7 namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)
