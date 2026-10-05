---
title: "CAP_SYS_ADMIN: container escape through the mount and admin capability"
description: "Escaping a container that holds CAP_SYS_ADMIN, the broad administrative capability that permits mount, cgroup, and namespace operations and so unlocks the cgroup release_agent escape, host filesystem mounts, and the sensitive procfs and sysfs writes."
keywords:
  - CAP_SYS_ADMIN
  - mount capability
  - cgroup release_agent
  - container escape
  - linux capabilities
---

# CAP_SYS_ADMIN

`CAP_SYS_ADMIN` is the catch-all administrative capability and the one most often added back. It permits `mount`, cgroup management, and many namespace operations, which is enough to escape on its own. The reliable route is the cgroup `release_agent` escape, which needs exactly this capability.

```bash
# Confirm it is effective
capsh --print | grep -q cap_sys_admin && echo have

# Mount a cgroup hierarchy and arm release_agent (see cgroups-release-agent for the full chain)
mkdir /tmp/cg && mount -t cgroup -o rdma cgroup /tmp/cg
```

## Exploitation notes

- The `mount` right also lets you remount a read-only `/proc/sys` writable and mount the host block device directly.
- It is the capability behind most sensitive `/proc` and `/sys` writes, so pair it with [procfs and sysfs](../../sensitive-mounts/procfs-and-sysfs/index.md).
- The full chain is in [cgroups release_agent](../cgroups-release-agent.md).

## References

- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [man 7 cgroups](https://man7.org/linux/man-pages/man7/cgroups.7.html)
