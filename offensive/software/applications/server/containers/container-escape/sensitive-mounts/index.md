---
title: "Sensitive mounts: escaping a container through dangerous bind mounts"
description: "Container escape through dangerous volume mounts: a host filesystem path that exposes the host's files directly, a runtime control socket that hands over the engine, or host procfs and sysfs paths that let the container write a program the host kernel then runs."
keywords:
  - sensitive mounts
  - host path mount
  - docker socket mount
  - procfs sysfs escape
  - container escape
---

# Sensitive mounts

A bind mount punches a hole in the container's filesystem view straight through to the host. Three kinds of mount turn that into a full escape: a host filesystem path gives direct read and write of host files, a runtime control socket gives command of the engine, and host procfs or sysfs paths expose kernel interfaces that run a program on the host. Enumerate what is mounted first, with `cat /proc/self/mountinfo` or `mount`.

## Subtopics

- **[Host path mount](host-path-mount.md)**: a host directory, often the whole root filesystem, bound into the container.
- **[Runtime socket mount](runtime-socket-mount.md)**: the engine's control socket (docker.sock, containerd.sock) bound into the container.
- **[procfs and sysfs](procfs-and-sysfs/index.md)**: writable host /proc and /sys paths that hand the kernel a program to run on the host.

## References

- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
