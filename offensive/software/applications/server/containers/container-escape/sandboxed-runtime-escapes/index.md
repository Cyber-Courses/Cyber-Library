---
title: "Sandboxed runtime escapes: breaking out of interposing sandboxes"
description: "Escaping sandboxed container runtimes that interpose on the kernel instead of sharing it directly: gVisor, which reimplements the kernel in userspace, and microVM runtimes such as Kata Containers, where the escape is from the sandbox or guest VM back to the host."
keywords:
  - gVisor
  - Kata Containers
  - sandbox escape
  - microVM
  - container escape
---

# Sandboxed runtime escapes

Sandboxed runtimes change the escape problem. Instead of sharing the host kernel directly, they put something in between: gVisor runs a userspace kernel that emulates syscalls, and Kata and other microVM runtimes run each workload in a lightweight virtual machine. A classic container escape does not apply; the target is a flaw in the sandbox's own interface back to the host.

## Subtopics

- **[gVisor](gvisor.md)**: escaping the userspace kernel and its host interface.
- **[Kata Containers](kata-containers.md)**: escaping the guest VM to the host.

## References

- [gVisor security model](https://gvisor.dev/docs/architecture_guide/security/)
- [Kata Containers architecture](https://katacontainers.io/learn/)
