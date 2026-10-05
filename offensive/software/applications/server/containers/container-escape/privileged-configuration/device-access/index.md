---
title: "Device access: escaping through host device nodes exposed in a container"
description: "When a container can see host device nodes under /dev, either from --privileged or an explicit --device, the kernel device drivers behind those nodes operate on host hardware regardless of namespaces. Mounting the host block device, reading raw kernel memory devices, or creating new device nodes with mknod each lead from the container to the host."
keywords:
  - container device access
  - dev directory
  - host block device
  - mknod
  - container escape
---

# Device access

Device nodes are the container's direct line to host hardware. A device node is only a reference to a driver identified by a major and minor number; the driver itself runs in the shared host kernel and acts on the real device, so a block-device node in a container reads the host's actual disk, not a namespaced copy. A container gets dangerous device nodes from `--privileged` (which adds a device cgroup rule allowing all devices) or from a targeted `--device=/dev/sda`, and in Kubernetes from a `volumeDevices` raw block claim.

Enumerate what is reachable:

```bash
ls -l /dev                              # long list with sdX/nvme/mem hints at broad access
cat /sys/fs/cgroup/devices/devices.list 2>/dev/null   # cgroup v1 device allowlist; "a *:* rwm" = all
lsblk 2>/dev/null; cat /proc/partitions # host block devices and sizes
```

A line of `a *:* rwm` in the device cgroup, or visible `sda`/`nvme0n1` nodes, means the host disk is one `mount` away.

## Subtopics

- **[Host block device](host-block-device.md)**: mount the host root filesystem and chroot in.
- **[Kernel memory devices](kernel-memory-devices.md)**: read and write RAM through /dev/mem and friends.
- **[mknod](mknod.md)**: recreate a device node when the path is missing but the capability is present.

## References

- [man 7 cgroups: device controller](https://man7.org/linux/man-pages/man7/cgroups.7.html)
- [Docker: --device and device cgroup rules](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
