---
title: "CAP_BPF: loading eBPF programs from a container"
description: "Abusing a container that holds CAP_BPF, and the capabilities that pair with it, to load eBPF programs that read kernel memory, trace and tamper with host activity, and in combination with known verifier flaws reach kernel code execution."
keywords:
  - CAP_BPF
  - eBPF
  - kernel memory
  - container escape
  - bpf verifier
---

# CAP_BPF

`CAP_BPF` (split out from `CAP_SYS_ADMIN` in newer kernels) lets the container load eBPF programs and create maps. eBPF runs in the kernel under a verifier, so it is a powerful observation and tampering primitive: helpers can read kernel memory and trace host activity, and verifier weaknesses have repeatedly led to kernel code execution.

```bash
capsh --print | grep -Eq 'cap_bpf|cap_sys_admin' && echo have
# Load a tracing program that reads kernel data (e.g. via bpftrace or a loader)
bpftrace -e 'kprobe:do_sys_openat2 { printf("%s %s\n", comm, str(arg1)) }' 2>/dev/null | head
```

## Exploitation notes

- Reading tracepoints and kernel structures leaks host secrets and addresses; attaching to syscalls observes every host process, not just the container's.
- For full escape, `CAP_BPF` typically pairs with `CAP_PERFMON` or `CAP_SYS_ADMIN`, and with a verifier bug it reaches arbitrary kernel read and write.
- Treat it like a kernel read primitive that also feeds a [kernel exploit](../../runtime-and-kernel-exploits/kernel-exploits/index.md).

## References

- [Kernel: BPF documentation](https://www.kernel.org/doc/html/latest/bpf/index.html)
- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
