---
title: "procfs and sysfs: escaping through writable host kernel interfaces"
description: "Container escape through host /proc and /sys paths bind-mounted into the container: writable kernel interfaces such as core_pattern, the modprobe path, uevent_helper, binfmt_misc, and sysrq-trigger that each cause the kernel to run an attacker-chosen program on the host, plus kcore and host process reads."
keywords:
  - sensitive mounts
  - core_pattern escape
  - uevent_helper
  - procfs sysfs
  - container escape
---

# procfs and sysfs

Several files under `/proc` and `/sys` are not data but control knobs: write a path into them and the kernel runs that program later, in the host's context, as root. These are safe only because a normal container does not get a writable host `/proc` or `/sys`. When one is bind-mounted in (a surprisingly common misconfiguration) or the container holds `CAP_SYS_ADMIN` in the initial namespaces, each knob becomes an escape. The catch shared by the executable-handler knobs is that the path must be reachable in the host filesystem namespace, so the helper is dropped on a host-visible path such as the container's overlay upperdir.

## Subtopics

- **[core_pattern](core_pattern.md)**: the program the kernel pipes core dumps to.
- **[modprobe path](modprobe-path.md)**: the helper the kernel runs to auto-load a module.
- **[uevent_helper](uevent-helper.md)**: the program the kernel runs on a device uevent.
- **[sysrq-trigger](sysrq-trigger.md)**: magic SysRq actions against the host.
- **[binfmt_misc](binfmt-misc.md)**: registering an interpreter for a file format.
- **[kcore memory read](kcore-memory-read.md)**: reading host kernel memory.
- **[Host process access](host-process-access.md)**: reading host processes through /proc.

## References

- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
