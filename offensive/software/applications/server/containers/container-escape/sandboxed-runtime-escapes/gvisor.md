---
title: "gVisor: escaping the userspace kernel and file proxy"
description: "gVisor isolates a container behind the Sentry, a userspace kernel that emulates Linux syscalls, and the Gofer, a process that proxies filesystem access. Escaping gVisor means finding a memory-safety or logic bug in the Sentry's syscall emulation, or abusing the Gofer file-proxy protocol, to execute in the Sentry or host context rather than inside the sandboxed guest."
keywords:
  - gvisor
  - sentry
  - gofer
  - syscall emulation
  - sandbox escape
---

# gVisor

gVisor does not let the container's syscalls reach the host kernel. Instead the Sentry, a kernel written in Go running in userspace, implements the Linux syscall interface itself, and a second process, the Gofer, serves filesystem requests over the 9P protocol so the Sentry never opens host files directly. The host kernel surface exposed to the Sentry is deliberately tiny and guarded by a strict seccomp filter. Escaping therefore means breaking the Sentry or the Gofer, not the host kernel: a bug in the Sentry's emulation of a syscall that yields memory corruption or an out-of-bounds access in the Sentry's address space, or a flaw in the Gofer protocol handling that lets the container reference host files outside the intended root.

Confirm gVisor and probe the emulated surface:

```bash
cat /proc/version                              # gVisor prints a characteristic string
dmesg 2>/dev/null | grep -i gvisor
# emulation gaps: syscalls that behave differently or are unimplemented
strace -f /bin/true 2>&1 | grep -i 'not implemented\|ENOSYS' | head
```

## Escape surfaces

- **Sentry syscall emulation**: the Sentry reimplements hundreds of syscalls in Go. A logic error or memory-safety slip in a complex one (futexes, splice, io_uring-style paths, or edge cases in filesystem and network emulation) can corrupt Sentry state. Code execution in the Sentry escapes the guest, after which the Sentry's own restricted host interface is the next barrier.
- **Gofer file proxy**: the Gofer answers 9P requests for the container's filesystem. A path-handling flaw that lets the container request a file outside its mapped root turns into host filesystem access, since the Gofer runs with more host access than the sandboxed guest.
- **Host interface after Sentry compromise**: even with code execution in the Sentry, a seccomp filter limits which host syscalls are allowed; a full host escape additionally needs a usable host-kernel bug reachable through that filtered surface.

## Exploitation notes

- gVisor shrinks and changes the attack surface rather than removing it: the target is Go code in the Sentry and the 9P Gofer protocol, so techniques differ sharply from native container escapes.
- Emulation incompleteness is a useful starting signal; syscalls that return unusual errors or behave differently from a native kernel mark areas where the implementation is doing non-trivial work and may hold bugs.
- Because the Sentry is memory-safe Go, pure memory-corruption is harder; logic bugs in file-path mapping and resource handling are historically the more productive class.

## References

- [gVisor security model](https://gvisor.dev/docs/architecture_guide/security/)
- [gVisor architecture: Sentry and Gofer](https://gvisor.dev/docs/architecture_guide/)
- [Google Project Zero: gVisor research](https://googleprojectzero.blogspot.com/)
