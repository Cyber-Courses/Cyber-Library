---
title: "Shared host namespaces: escaping through a shared host namespace"
description: "Container escape and pivoting when the container shares one of the host's namespaces: host PID for nsenter into host processes, host network for the host's interfaces and loopback services, host IPC for shared memory, and host user namespace where container root is real host root."
keywords:
  - host namespace
  - hostPID
  - hostNetwork
  - nsenter
  - container escape
---

# Shared host namespaces

Namespaces are what separate a container from the host. Sharing any of the host's namespaces back (`--pid=host`, `--net=host`, `--ipc=host`, `userns=host`, or the Kubernetes `hostPID`/`hostNetwork`/`hostIPC`) removes one wall. Host PID is the strongest: it exposes every host process for `nsenter` or injection. Check what is shared:

```bash
ls -l /proc/1/ns/            # compare namespace inode numbers with the host's
cat /proc/1/comm             # if this is the host's init, PID namespace is shared
```

## Subtopics

- **[Host PID namespace](host-pid-namespace.md)**: see host processes and nsenter into them.
- **[Host network namespace](host-network-namespace.md)**: the host's interfaces and loopback services.
- **[Host IPC namespace](host-ipc-namespace.md)**: host shared memory and IPC objects.
- **[Host user namespace](host-user-namespace.md)**: container root is real host root.

## References

- [man 7 namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)
- [man 1 nsenter](https://man7.org/linux/man-pages/man1/nsenter.1.html)
