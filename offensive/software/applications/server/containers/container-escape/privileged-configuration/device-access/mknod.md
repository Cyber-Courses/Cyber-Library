---
title: "mknod: creating a device node to reach a host device"
description: "Escaping a container that holds CAP_MKNOD by creating a device node for a host block device that was not mapped into the container, then reading or mounting it to reach the host filesystem."
keywords:
  - mknod
  - CAP_MKNOD
  - device node
  - container escape
  - host disk
---

# mknod

`CAP_MKNOD` lets the container create device nodes. Because a device node is just a major and minor number, the container can create a node for a host disk that was never granted to it, then read or mount that device.

```bash
capsh --print | grep -q cap_mknod && echo have
# Recreate the host root block device by its major:minor and read it
mknod /dev/hostsda b 8 1 && dd if=/dev/hostsda bs=1M count=1 2>/dev/null | file -
```

## Exploitation notes

- The default Docker cgroup device policy denies access to unlisted devices even if the node exists, so `mknod` escapes work where that policy is loosened or in a privileged container.
- Determine the target major:minor from a readable `/proc/partitions` on the host or from known defaults (8:0 for the first SCSI/SATA disk).
- Once the node is readable, continue as in [Host block device](host-block-device.md).

## References

- [man 2 mknod](https://man7.org/linux/man-pages/man2/mknod.2.html)
- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
