---
title: "Privileged configuration: escaping through over-permissive container settings"
order: 2
description: "Container escape through configuration that hands the workload too much: the all-in-one privileged flag, individual dangerous Linux capabilities, direct host device access, a disabled seccomp or AppArmor profile, and a writable cgroup release_agent. Each is a concrete, no-exploit route from inside a container to code execution on the host."
keywords:
  - privileged container
  - linux capabilities
  - seccomp apparmor
  - cgroup release_agent
  - container escape
---

# Privileged configuration

The most common container escapes need no memory-corruption exploit at all: the container was simply started with more power than isolation can survive, and the attacker uses that power as designed. Before trying anything, fingerprint exactly what you were given, because it decides which route works.

```bash
# Effective capability set (the single most important check)
grep CapEff /proc/self/status
# => CapEff: 000001ffffffffff  is the FULL set = --privileged; decode any value with:
capsh --decode=000001ffffffffff

# Seccomp mode: 0 = disabled, 2 = a filter is loaded
grep Seccomp /proc/self/status
# AppArmor/SELinux confinement for this process
cat /proc/self/attr/current 2>/dev/null        # "unconfined" means no AppArmor profile

# Device access and the cgroup version/writability
ls -l /dev | head                                # a long device list hints at --privileged
cat /proc/1/cgroup; mount | grep cgroup          # v1 (named hierarchies) vs v2 (unified)

# User namespace: is container root real host root?
cat /proc/self/uid_map                           # "0 0 4294967295" => UID 0 == host UID 0
```

Read that output top to bottom: a full `CapEff`, a visible device list, and `Seccomp: 0` together mean `--privileged`, and every route below is open. A single added capability narrows you to its specific primitive. A `0 0 ...` uid_map means whatever you achieve runs as real host root; a mapped range confines it.

## Subtopics

- **[Privileged flag](privileged-flag.md)**: `--privileged`, which grants everything below at once.
- **[Capability abuse](capability-abuse/index.md)**: a single dangerous capability, each with its own route to the host.
- **[Device access](device-access/index.md)**: raw host devices exposed in the container.
- **[Unconfined seccomp or AppArmor](unconfined-seccomp-or-apparmor.md)**: a weakened syscall and LSM profile.
- **[cgroups release_agent](cgroups-release-agent.md)**: a writable release_agent that runs a program on the host.

## References

- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Docker: runtime privilege and Linux capabilities](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
