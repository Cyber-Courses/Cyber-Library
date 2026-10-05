---
title: "Device access: escaping through host devices exposed in a container"
description: "Container escape through direct access to host devices: reading or mounting a raw host block device to reach the host filesystem, reading and writing kernel or physical memory through /dev/mem, or creating a device node with mknod to reach a device that was not mapped in."
keywords:
  - device access
  - /dev/sda
  - /dev/mem
  - mknod
  - container escape
---

# Device access

A container that can reach host devices escapes through them. A privileged container sees every device; a specific `--device` grant or `CAP_MKNOD` exposes one. The two that matter most are the host's root block device (mount it, own the filesystem) and kernel or physical memory (patch the running kernel).

```bash
ls -l /dev                      # what devices are visible
cat /proc/partitions            # host block devices, e.g. sda
```

## Subtopics

- **[Host block device](host-block-device.md)**: mount the host disk directly.
- **[Kernel memory devices](kernel-memory-devices.md)**: read and write memory through /dev/mem.
- **[mknod](mknod.md)**: create a device node to reach a host device.

## References

- [man 4 mem](https://man7.org/linux/man-pages/man4/mem.4.html)
- [Docker runtime privilege and capabilities](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)
