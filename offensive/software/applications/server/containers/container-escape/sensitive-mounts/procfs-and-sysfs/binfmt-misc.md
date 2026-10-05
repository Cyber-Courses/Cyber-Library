---
title: "binfmt_misc: host code execution through a registered interpreter"
description: "Escaping a container by registering an interpreter through /proc/sys/fs/binfmt_misc, so that executing a file matching the registered magic or extension runs the attacker-chosen interpreter on the host."
keywords:
  - binfmt_misc
  - binary format handler
  - procfs escape
  - container escape
  - interpreter registration
---

# binfmt_misc

`binfmt_misc` lets the kernel run a chosen interpreter for files matching a magic byte sequence or extension. Its control file, `/proc/sys/fs/binfmt_misc/register`, registers new handlers, and the interpreter path is resolved and executed in the host context. A container with a writable host `binfmt_misc` mount registers a handler pointing at a host-visible interpreter, then triggers it.

```bash
# Register a handler: name, type (M=magic), offset, magic, mask, interpreter, flags
host_path=$(sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
printf ':esc:M::\\x7fRUN::%s/x:' "$host_path" > /proc/sys/fs/binfmt_misc/register

# Any process that execs a file starting with those magic bytes now runs /x on the host
```

## Exploitation notes

- The `F` (fix binary) flag makes the kernel open the interpreter at registration time, which helps when the container's view of the path differs from the host's.
- Registration requires the `binfmt_misc` filesystem to be mounted and writable, which only happens with a privileged or deliberately misconfigured container.
- Like the other handler knobs, the interpreter runs on the host; see [core_pattern](core_pattern.md) for the same pattern.

## References

- [Kernel: binfmt_misc](https://www.kernel.org/doc/html/latest/admin-guide/binfmt-misc.html)
- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
