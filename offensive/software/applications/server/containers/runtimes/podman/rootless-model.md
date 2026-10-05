---
title: "Rootless model: the rootless Podman user-namespace mapping and its limits"
description: "Understanding the rootless Podman model for attackers: container root maps to the invoking user on the host through subuid and subgid, so an escape lands as that unprivileged user, not host root, which changes which container-escape primitives are worth trying."
keywords:
  - rootless podman
  - subuid subgid
  - user namespace
  - rootless escape
  - container security
---

# Rootless model

Rootless Podman runs the whole stack as an ordinary user, using a user namespace where container UID 0 maps to the invoking user and a range of `subuid`/`subgid` values. For an attacker this reframes the escape: breaking out lands you as that unprivileged host user, not root, so the goal shifts to what that user can reach and to local privilege escalation from there.

```bash
cat /proc/self/uid_map                     # 0 <hostuid> 1 then the subuid range
cat /etc/subuid /etc/subgid                # the mapped ranges for the user
podman unshare cat /proc/self/uid_map      # view inside the podman user namespace
```

## Exploitation notes

- Capability-based escapes still work inside the namespace but apply only to resources the mapped user owns, so they do not reach host root directly.
- The realistic path is escape to the host user, then ordinary Linux local privilege escalation; treat the rootless container as a foothold as that user.
- A rootful Podman (run as root or via the system socket) does not have this limit; confirm the mapping before assuming confinement.

## References

- [Podman rootless containers](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)
- [man 7 user_namespaces](https://man7.org/linux/man-pages/man7/user_namespaces.7.html)
