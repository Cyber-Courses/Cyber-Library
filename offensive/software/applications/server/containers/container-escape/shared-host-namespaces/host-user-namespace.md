---
title: "Host user namespace: container root as real host root"
description: "Why a container that shares the host user namespace is more dangerous: UID 0 inside the container is UID 0 on the host, so every capability the container holds applies against host resources without the mapping that would otherwise neuter it."
keywords:
  - host user namespace
  - userns
  - rootless containers
  - container escape
  - UID mapping
---

# Host user namespace

User namespaces let a container's root be an unprivileged user on the host, which is what makes rootless containers safer. Running without one (`userns=host`, the default for rootful Docker and most Kubernetes pods) means container UID 0 is host UID 0, so capabilities and file access apply with full force against host resources. It is less a standalone escape than the precondition that makes the others work.

```bash
cat /proc/self/uid_map
# 0 0 4294967295  => no mapping, container root IS host root
# 0 100000 65536  => mapped, container root is an unprivileged host user
```

## Exploitation notes

- A `uid_map` of `0 0 ...` is the signal: every capability-based escape ([Capability abuse](../privileged-configuration/capability-abuse/index.md)) and every host-file write acts as real root.
- Under a real user-namespace mapping, the same capabilities are confined to the mapped range, so many escapes fail or are limited to the namespace.
- When a mapping is present, look instead for mapping overlaps or a shared `/proc`/`/sys` that crosses it.

## References

- [man 7 user_namespaces](https://man7.org/linux/man-pages/man7/user_namespaces.7.html)
- [Docker: isolate containers with a user namespace](https://docs.docker.com/engine/security/userns-remap/)
