---
title: "CAP_SYS_PTRACE: injecting code into host processes across a shared PID namespace"
description: "CAP_SYS_PTRACE allows a container to trace and manipulate processes it can see. Combined with a shared host PID namespace (--pid=host), it lets an attacker attach to a host process, write shellcode into its memory through /proc/pid/mem or PTRACE_POKETEXT, and hijack its execution to run code as that process's user, typically root."
keywords:
  - cap_sys_ptrace
  - ptrace
  - pid namespace
  - process injection
  - container escape
---

# CAP_SYS_PTRACE

`CAP_SYS_PTRACE` lets a process trace any other process it can see, bypassing the usual same-UID and `Yama` ptrace restrictions. By itself inside a container it only reaches the container's own processes. The escape appears when the container also shares the host PID namespace, which `docker run --pid=host` and Kubernetes `hostPID: true` grant: now every host process is visible, and the capability lets the attacker inject into one of them and execute code in the host context.

Confirm both preconditions:

```bash
capsh --print | grep -o cap_sys_ptrace
grep CapEff /proc/self/status          # bit 19 set
ls /proc | grep -E '^[0-9]+$' | head   # host PIDs visible => shared PID namespace
ps aux | grep -v grep | grep -E 'sshd|cron|systemd'   # pick a long-lived root target
```

## Route: write to /proc/pid/mem

The simplest modern injection avoids raw `ptrace` opcodes. With `CAP_SYS_PTRACE` you can `PTRACE_SEIZE` a target, read its registers, and overwrite instructions at the current `rip` by writing to `/proc/<pid>/mem`, then let it run your shellcode.

```c
// outline: hijack a host process to run execve("/bin/sh", ...)
ptrace(PTRACE_SEIZE, target_pid, 0, 0);
ptrace(PTRACE_INTERRUPT, target_pid, 0, 0);
struct user_regs_struct regs; ptrace(PTRACE_GETREGS, target_pid, 0, &regs);
int fd = open("/proc/<pid>/mem", O_RDWR);
pwrite(fd, shellcode, sizeof(shellcode), regs.rip);   // overwrite at instruction pointer
ptrace(PTRACE_SETREGS, target_pid, 0, &regs);
ptrace(PTRACE_DETACH, target_pid, 0, 0);              // target executes shellcode
```

The shellcode runs with the target's privileges; choosing a root-owned host daemon yields host root. A reverse-shell or `authorized_keys`-writing payload is typical.

## Route: classic POKETEXT

Where `/proc/pid/mem` writes are blocked, the older path is `PTRACE_ATTACH`, then `PTRACE_POKETEXT` word by word to patch the text segment, then `PTRACE_SETREGS` and `PTRACE_DETACH`. It is slower but relies only on the ptrace syscall surface the capability unlocks.

## Exploitation notes

- No shared PID namespace means no host processes in `/proc`, and the capability only reaches container-internal processes. Check `ls /proc` for high host-assigned PIDs or `readlink /proc/1/exe` pointing at the host init.
- A seccomp filter that blocks `ptrace` neutralises this even when the capability is present; test `strace /bin/true` or a minimal `ptrace(PTRACE_TRACEME)` call.
- Prefer injecting into a stable, frequently-idle root daemon (for example `cron`) rather than a busy process, to avoid visibly disrupting a service while you gain execution.

## References

- [man 2 ptrace](https://man7.org/linux/man-pages/man2/ptrace.2.html)
- [HackTricks: CAP_SYS_PTRACE](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_sys_ptrace)
