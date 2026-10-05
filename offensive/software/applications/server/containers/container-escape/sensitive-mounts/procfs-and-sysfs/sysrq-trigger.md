---
title: "sysrq-trigger: driving host kernel magic-sysrq actions from a container"
description: "Writing a character to /proc/sysrq-trigger invokes the kernel's magic-sysrq handler on the host: syncing disks, killing processes, thawing filesystems, or crashing and rebooting the machine. A container with the host procfs mounted writable uses it to disrupt the host or to combine process kills with other primitives, though its direct code-execution value is limited."
keywords:
  - sysrq-trigger
  - magic sysrq
  - proc sysrq
  - denial of service
  - container escape
---

# sysrq-trigger

`/proc/sysrq-trigger` is a write-only entry point to the kernel's magic-sysrq handler. A single character written to it runs the corresponding sysrq command against the host kernel: `s` syncs disks, `e` sends SIGTERM to all processes except init, `i` sends SIGKILL to all, `f` invokes the out-of-memory killer, `c` crashes the kernel, and `b` reboots immediately without unmounting. These act on the host because sysrq is global, so a container with the host procfs mounted writable can reach host-wide effects.

Confirm access and that sysrq is enabled:

```bash
[ -w /proc/sysrq-trigger ] && echo writable
cat /proc/sys/kernel/sysrq          # bitmask of allowed sysrq functions; 1 = all
```

## Available actions

```bash
echo s > /proc/sysrq-trigger        # sync all mounted filesystems
echo e > /proc/sysrq-trigger        # SIGTERM to all user processes (mass kill)
echo i > /proc/sysrq-trigger        # SIGKILL to all user processes
echo c > /proc/sysrq-trigger        # crash the host (kernel panic)
echo b > /proc/sysrq-trigger        # immediate reboot, no clean unmount
```

## Exploitation notes

- This is primarily a denial-of-service and disruption primitive, not a direct code-execution escape: it cannot run an attacker program the way [core_pattern](core_pattern.md) or [uevent_helper](uevent-helper.md) can.
- The mass-kill actions (`e`, `i`) can be chained: killing a host process can force a supervisor to restart it in a way the attacker influences, or clear a lock an attacker needs, but on its own it only destroys availability.
- The `sysrq` bitmask in `/proc/sys/kernel/sysrq` may restrict which functions are permitted; `0` disables sysrq entirely and defeats the technique.
- Reaching this needs the host procfs mounted writable, typically a privileged container.

## References

- [Kernel docs: magic SysRq key](https://docs.kernel.org/admin-guide/sysrq.html)
- [HackTricks: sensitive mounts](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
