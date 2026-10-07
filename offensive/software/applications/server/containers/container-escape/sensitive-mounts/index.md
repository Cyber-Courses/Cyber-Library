---
title: "Sensitive mounts: escaping through host paths and kernel interfaces bind-mounted into a container"
order: 1
description: "Container escapes that come from a host path exposed inside the container: a bind mount of a host directory or the full host root, a mounted runtime control socket such as docker.sock, and writable procfs or sysfs kernel interfaces like core_pattern, modprobe, and uevent_helper that the host kernel executes as root."
keywords:
  - sensitive mounts
  - docker socket
  - core_pattern
  - hostpath
  - container escape
---

# Sensitive mounts

Many escapes need no capability trick at all: the container was simply given a host path it should not have. That path can be an explicit directory bind mount, the whole host root, a runtime control socket, or a kernel pseudo-file under `/proc` and `/sys` that the host kernel reads back and acts on as root. The common thread is that the object on the other side of the mount belongs to the host, so writing it reaches the host directly.

Enumerate mounts and look for host-owned sources:

```bash
cat /proc/self/mountinfo                 # full mount table with source subtrees
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS  # readable summary; host bind sources stand out
# control sockets and sensitive kernel files
ls -l /var/run/docker.sock /run/containerd/containerd.sock /run/crio/crio.sock 2>/dev/null
ls -l /proc/sys/kernel/core_pattern /proc/sys/kernel/modprobe /sys/kernel/uevent_helper 2>/dev/null
mount | grep -E '/proc|/sys' | grep -v 'ro,'   # writable proc/sys indicates host interfaces
```

A host bind mount shows a host subtree as its source; a writable `/proc/sys/kernel/*` or a runtime socket is a direct route.

## Subtopics

- **[Host path mount](host-path-mount.md)**: a bind mount of a host directory or the root filesystem.
- **[Runtime socket mount](runtime-socket-mount.md)**: a mounted Docker, containerd, or CRI-O control socket.
- **[procfs and sysfs](procfs-and-sysfs/index.md)**: writable kernel interfaces the host executes as root.

## References

- [BishopFox: bad pod and sensitive mounts](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [HackTricks: sensitive mounts](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
