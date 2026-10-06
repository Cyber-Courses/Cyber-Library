---
title: "Shared host namespaces: escapes from a container that shares the host's namespaces"
order: 4
description: "Each --host namespace flag removes one layer of isolation between container and host. Sharing the PID namespace exposes and lets you inject into host processes, the network namespace exposes host-local services and the node's loopback, the IPC namespace exposes shared memory and semaphores, and a shared or mapped user namespace can mean container root is host root."
keywords:
  - host namespace
  - hostpid hostnetwork
  - pid namespace
  - user namespace
  - container escape
---

# Shared host namespaces

Namespaces are what make a container a container: separate views of processes, network, IPC, mounts, users, and more. Every `--pid=host`, `--net=host`, `--ipc=host`, or `--userns=host` flag (and the Kubernetes `hostPID`, `hostNetwork`, `hostIPC` fields) removes one of those separations, re-joining the container to the host's view. None of these is a memory-corruption exploit; each simply hands back a slice of the host, and the escape is using that slice.

Fingerprint which namespaces are shared with the host by comparing namespace IDs against PID 1 on the host, or by what is visible:

```bash
ls -l /proc/self/ns/                       # this process's namespace inode IDs
ls -l /proc/1/ns/                          # compare: same inode = shared with host init
ls /proc | grep -E '^[0-9]+$' | wc -l      # many host PIDs => shared PID namespace
ip addr; ss -tlnp 2>/dev/null | head       # host interfaces/ports => shared net namespace
ipcs                                       # host shared memory/semaphores => shared IPC
cat /proc/self/uid_map                     # "0 0 ..." => not in a user namespace
```

## Subtopics

- **[Host PID namespace](host-pid-namespace.md)**: see and inject into host processes.
- **[Host network namespace](host-network-namespace.md)**: reach host-local services and intercept traffic.
- **[Host IPC namespace](host-ipc-namespace.md)**: read shared memory and message queues.
- **[Host user namespace](host-user-namespace.md)**: when container root maps to host root.

## References

- [man 7 namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)
- [Kubernetes: pod security and host namespaces](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [HackTricks: namespaces](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces)
