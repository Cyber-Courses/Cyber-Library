---
title: "procfs and sysfs: kernel pseudo-files the host executes as root"
description: "Several files under /proc/sys and /sys are usermode-helper paths or triggers: the host kernel reads them and runs the referenced program as root in the host namespaces, or exposes host memory. When a container has the real host procfs or sysfs mounted writable, writing core_pattern, modprobe, or uevent_helper, or reading kcore, leads straight to the host."
keywords:
  - procfs sysfs
  - usermode helper
  - core_pattern
  - uevent_helper
  - container escape
---

# procfs and sysfs

`/proc` and `/sys` are not ordinary files: many entries are control points the kernel reads back and acts on. A handful name a program the kernel runs, as root in the host's init namespaces, when some event occurs: a crash (`core_pattern`), an auto-load of a kernel module (`modprobe`), or a device uevent (`uevent_helper`). Others expose host memory directly (`kcore`) or let a process poke the host kernel (`sysrq-trigger`). These are escapes only when the container has the host's real procfs or sysfs mounted and writable, which happens with a careless `-v /proc:/host/proc`, a procfs remount, or a privileged container.

Check what is exposed and writable:

```bash
mount | grep -E 'proc|sys'                       # look for host proc/sys without "ro"
ls -l /proc/sys/kernel/core_pattern /proc/sys/kernel/modprobe 2>/dev/null
ls -l /sys/kernel/uevent_helper 2>/dev/null
for f in /proc/sys/kernel/core_pattern /proc/sys/kernel/modprobe /sys/kernel/uevent_helper; do
  [ -w "$f" ] && echo "writable: $f"; done
```

## Subtopics

- **[core_pattern](core_pattern.md)**: pipe a crashing process's core dump to a host program.
- **[modprobe path](modprobe-path.md)**: hijack the module auto-loader the kernel runs as root.
- **[uevent_helper](uevent-helper.md)**: run a program on a device uevent.
- **[binfmt_misc](binfmt-misc.md)**: register an interpreter for a file format.
- **[sysrq-trigger](sysrq-trigger.md)**: invoke host kernel magic-sysrq actions.
- **[host process access](host-process-access.md)**: read host process memory and environment.
- **[kcore memory read](kcore-memory-read.md)**: dump host kernel memory through /proc/kcore.

## References

- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
- [HackTricks: sensitive mounts](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
- [Kernel docs: call_usermodehelper interfaces](https://docs.kernel.org/admin-guide/sysctl/kernel.html)
