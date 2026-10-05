---
title: "Rootless model: what a user-namespace runtime changes for the attacker"
description: "Rootless Podman runs containers inside a user namespace where container root maps to the invoking unprivileged user on the host. A container escape therefore yields that user's privileges, not host root, and capabilities apply only within the namespace. Understanding the uid mapping decides which escapes are worth attempting and what they achieve."
keywords:
  - rootless podman
  - user namespace
  - uid_map
  - subuid subgid
  - container escape
---

# Rootless model

Rootless Podman runs entirely as an unprivileged user by placing containers in a user namespace. Inside, the container sees UID 0, but that maps through `/etc/subuid` and `/etc/subgid` to a range of unprivileged host UIDs. The consequence for an attacker is decisive: a container escape lands as the invoking user, not as host root, and the capabilities a container process holds are valid only against resources owned by the mapped range. Many of the capability and device escapes that give host root under rootful Docker give only a confined, remapped identity here.

Read the mapping to know what an escape yields:

```bash
cat /proc/self/uid_map
#   0     100000     65536   => container root maps to host UID 100000..165535
id; podman info | grep -i rootless
grep "^$USER:" /etc/subuid /etc/subgid                # the allocated ranges
```

## What still works, and what it gives

```bash
# Escaping the container (e.g. via a bad mount) gives the invoking user's access,
# so target that user's secrets, SSH keys, and reachable services, not /etc/shadow.
ls -la $HOME/.ssh $HOME/.config 2>/dev/null
# Capability routes act only within the user namespace: CAP_SYS_ADMIN here cannot
# write host-root-owned files, because the kernel checks ownership against the ns.
# A genuine host-root escape needs a user-namespace-aware kernel bug.
```

## Exploitation notes

- Always read `uid_map` first; a mapped range (not `0 0`) means capability and device escapes yield the mapped user, so prioritise the invoking user's own assets over host-root targets.
- Rootless does expand the kernel attack surface in one way: creating the user namespace grants capabilities inside it, which can reach subsystems (netfilter, some filesystems) for a kernel exploit that does escalate to host root; see [nf_tables](../../container-escape/runtime-and-kernel-exploits/kernel-exploits/nf-tables.md).
- If the user Podman runs as is itself privileged elsewhere (a member of sensitive groups, holding cloud credentials), escaping to that user is still a strong position even without host root.

## References

- [Podman: rootless tutorial](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)
- [Rootless containers](https://rootlesscontaine.rs/)
- [man 7 user_namespaces](https://man7.org/linux/man-pages/man7/user_namespaces.7.html)
