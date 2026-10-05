---
title: "Host IPC namespace: reaching host shared memory from a container"
description: "Abusing a container that shares the host IPC namespace to read and write host shared memory segments and SysV or POSIX IPC objects, leaking secrets held in shared memory or interfering with host processes that rely on it."
keywords:
  - hostIPC
  - shared memory
  - System V IPC
  - container escape
  - inter-process communication
---

# Host IPC namespace

Sharing the host IPC namespace (`--ipc=host`, Kubernetes `hostIPC`) gives the container the host's System V and POSIX IPC objects: shared memory segments, semaphores, and message queues. Processes that keep secrets or state in shared memory (some databases, caches, and agents) expose it to the container.

```bash
ipcs                                     # host shared memory segments, queues, semaphores
# Attach to and dump a host shared-memory segment by its id
ipcs -m; cat /proc/sysvipc/shm
```

## Exploitation notes

- It is primarily an information and interference primitive, not a direct root escape: read secrets from shared memory or corrupt a host process's shared state.
- The value depends on what host software uses shared memory; pair it with [Host PID namespace](host-pid-namespace.md) for a stronger foothold.
- POSIX shared memory also surfaces under `/dev/shm`, which may be shared separately.

## References

- [man 7 svipc](https://man7.org/linux/man-pages/man7/svipc.7.html)
- [man 7 namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)
