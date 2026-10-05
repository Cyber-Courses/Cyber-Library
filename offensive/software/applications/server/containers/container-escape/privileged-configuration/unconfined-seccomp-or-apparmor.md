---
title: "Unconfined seccomp or AppArmor: escaping through weakened confinement"
description: "Escaping or widening a container's attack surface when its seccomp or AppArmor profile is disabled or weakened, restoring dangerous syscalls such as mount, ptrace, keyctl, and unshare that the default profiles block and that other escapes depend on."
keywords:
  - seccomp unconfined
  - apparmor unconfined
  - container escape
  - syscall filter
  - ptrace mount
---

# Unconfined seccomp or AppArmor

The default seccomp and AppArmor profiles are a large part of what keeps a container contained: they block or restrict syscalls like `mount`, `ptrace`, `keyctl`, `unshare`, `bpf`, and `perf_event_open`. Running with `--security-opt seccomp=unconfined` or `apparmor=unconfined` hands those back, which rarely escapes on its own but unlocks the techniques that do.

```bash
# Detect the posture
grep Seccomp /proc/self/status        # 0 = disabled, 2 = filtered
cat /proc/self/attr/current           # AppArmor profile, "unconfined" if off

# With seccomp off, syscalls the default profile blocks now work, e.g. mount and ptrace
unshare -m 2>&1 | head -1
```

## Exploitation notes

- Unconfined seccomp is the enabler for capability-based escapes that call `mount`, for `ptrace` injection into host processes, and for `keyctl` and `bpf` abuse; combine it with the matching capability.
- AppArmor unconfined removes the path and mount restrictions Docker's default profile adds, which is what blocks writing to sensitive `/proc` paths in a default container.
- Treat a profile that is off as a precondition, then reach for [Capability abuse](capability-abuse/index.md) or [procfs and sysfs](../sensitive-mounts/procfs-and-sysfs/index.md).

## References

- [Docker seccomp security profiles](https://docs.docker.com/engine/security/seccomp/)
- [Docker AppArmor security profiles](https://docs.docker.com/engine/security/apparmor/)
