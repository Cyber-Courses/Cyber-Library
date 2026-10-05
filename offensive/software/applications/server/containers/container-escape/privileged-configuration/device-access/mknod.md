---
title: "mknod: recreating a host device node inside the container"
description: "Even when a container's /dev does not list a sensitive device, a process holding CAP_MKNOD and knowing the device's major and minor numbers can recreate the node with mknod and then use it. This reconstructs access to the host disk or raw memory when the device cgroup still permits the device but the node was not created."
keywords:
  - mknod
  - cap_mknod
  - device major minor
  - device node
  - container escape
---

# mknod

A device node carries no data of its own: it is just a filesystem entry tagging a major and minor number that the kernel resolves to a driver. `mknod` creates such an entry. If the container holds `CAP_MKNOD` (part of the Docker default set) and the device cgroup still allows a device, an attacker can recreate the node for the host disk or a raw memory device even though the image's `/dev` did not include it, then use that node to reach the host.

Check the capability and what the device cgroup permits:

```bash
capsh --print | grep -o cap_mknod
cat /sys/fs/cgroup/devices/devices.list 2>/dev/null
# "b *:* rwm" allows all block devices; "c 1:1 rwm" would allow /dev/mem
```

The device cgroup `devices.list` is the real gate: `mknod` succeeds in creating the node regardless, but opening it is denied unless the cgroup allows that major/minor. A permissive list (common under `--privileged`, which sets `a *:* rwm`) means any recreated node works.

## Recreate and use a block device

```bash
# host root disk is typically major 8 (sd*) or 259 (nvme); minor from /proc/partitions
cat /proc/partitions                     # find major:minor of the host root partition
mknod /tmp/hostdisk b 8 1                # recreate /dev/sda1 as a block node
mount /tmp/hostdisk /mnt/host 2>/dev/null || \
  debugfs -R 'cat /etc/shadow' /tmp/hostdisk   # read without mounting
```

For a raw memory device:

```bash
mknod /tmp/mem c 1 1                      # /dev/mem is char major 1 minor 1
```

## Exploitation notes

- `mknod` creating the node is necessary but not sufficient; the device cgroup allowlist decides whether opening it is permitted. Read `devices.list` first to know which majors/minors are usable.
- Discover major/minor numbers from `/proc/partitions` for block devices and from the kernel's `Documentation/admin-guide/devices.txt` conventions for character devices (mem is 1:1, port is 1:4).
- This is the enabler when a device route is theoretically open (permissive cgroup) but the node is missing; combine with [Host block device](host-block-device.md) or [Kernel memory devices](kernel-memory-devices.md) for the actual read/write.

## References

- [man 2 mknod](https://man7.org/linux/man-pages/man2/mknod.2.html)
- [Linux devices.txt (major/minor assignments)](https://www.kernel.org/doc/Documentation/admin-guide/devices.txt)
- [HackTricks: CAP_MKNOD](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_mknod)
