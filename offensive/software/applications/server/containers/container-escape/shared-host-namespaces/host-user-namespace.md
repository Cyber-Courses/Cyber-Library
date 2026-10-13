---
title: "Host user namespace: when container root is host root"
order: 4
description: "If a container is not in a separate user namespace, UID 0 inside it is UID 0 on the host, so any capability or file access achieved in the container applies with real host privilege. Sharing or not using a user namespace is what makes the other escape routes yield actual host root rather than a confined, remapped identity."
keywords:
  - user namespace
  - uid_map
  - host root
  - rootful container
  - container escape
---

# Host user namespace

The user namespace decides whether container root is real root. When a container runs without its own user namespace (the default for rootful Docker and most Kubernetes pods), UID 0 inside the container is UID 0 on the host, and the capabilities a process holds are host capabilities. When a user namespace is in use, container UID 0 maps to an unprivileged host UID, and capabilities are valid only inside that namespace, so the same escape primitive yields a remapped, confined identity rather than host root. This page is about recognising which situation you are in, because it determines whether every other route on these pages actually gives host root.

Read the mapping:

```bash
cat /proc/self/uid_map
# "         0          0 4294967295"  => identity map: container UID 0 == host UID 0
# "         0     100000      65536"  => user namespace: container 0 == host 100000
cat /proc/self/gid_map
id
```

An identity `uid_map` starting `0 0` means no isolating user namespace: you are operating as genuine host root once you reach the host. A mapped range means a user namespace is active and your root is remapped.

## Why it matters for the other routes

```bash
# With an identity map, a capability-based escape runs as host root:
#   CAP_SYS_MODULE -> module loads into the host kernel as real root
#   a host disk mount -> files written are owned by real host root
# With a mapped user namespace, the same CAP_* are namespaced:
#   capabilities apply only to resources owned by the mapped range,
#   and host files owned by real root remain unwritable
```

A user namespace does not stop you from, for example, reading world-readable secrets, but it blocks the capability routes from acting on host-owned resources, because the kernel checks the capability against the resource's owning namespace.

## Exploitation notes

- Always check `uid_map` before investing in a capability or mount route: an identity map confirms the route ends at host root; a mapped range means you may gain only a confined, remapped identity.
- Rootless Docker and Podman use a user namespace by default, which is why their escapes are harder and often require a user-namespace-aware bug rather than a configuration abuse.
- Some host escapes (for example a runtime-socket mount that launches a new rootful container) bypass the user-namespace limitation entirely, because the host daemon, not the confined process, creates the privileged container.

## References

- [man 7 user_namespaces](https://man7.org/linux/man-pages/man7/user_namespaces.7.html)
- [Rootless containers](https://rootlesscontaine.rs/)
- [HackTricks: user namespace](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/user-namespace)
