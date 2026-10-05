---
title: "Privileged configuration: escaping through over-permissive container settings"
description: "Container escape through configuration that hands the workload too much: the all-in-one privileged flag, a single dangerous Linux capability, direct host device access, a disabled seccomp or AppArmor profile, or a writable cgroup release_agent."
keywords:
  - privileged container
  - linux capabilities
  - seccomp apparmor
  - cgroup release_agent
  - container escape
---

# Privileged configuration

The most common container escapes need no exploit at all: the container was simply configured with more power than isolation can survive. Each setting here removes one of the walls, and several of them collapse straight to host code execution. Check what you were given first:

```bash
capsh --print 2>/dev/null                 # capabilities in the container
cat /proc/self/status | grep -i cap       # CapEff bitmask
cat /sys/fs/cgroup/*/release_agent 2>/dev/null
```

## Subtopics

- **[Privileged flag](privileged-flag.md)**: the all-in-one `--privileged`, which grants everything below at once.
- **[Capability abuse](capability-abuse/index.md)**: a single dangerous capability, each with its own route to the host.
- **[Device access](device-access/index.md)**: raw host devices exposed in the container.
- **[Unconfined seccomp or AppArmor](unconfined-seccomp-or-apparmor.md)**: a weakened or disabled syscall and LSM profile.
- **[cgroups release_agent](cgroups-release-agent.md)**: a writable release_agent that runs a program on the host.

## References

- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Docker runtime privilege and capabilities](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)
