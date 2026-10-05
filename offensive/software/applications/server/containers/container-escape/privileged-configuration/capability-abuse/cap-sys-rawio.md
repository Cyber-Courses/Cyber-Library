---
title: "CAP_SYS_RAWIO: patching the kernel through raw port and physical memory access"
description: "CAP_SYS_RAWIO grants ioperm, iopl, and access to /dev/mem, /dev/kmem, and /dev/port. Inside a container that also exposes those device nodes, it lets an attacker read and write physical memory directly, locate kernel structures, and overwrite them to disable protections or elevate a process, bypassing namespace isolation entirely."
keywords:
  - cap_sys_rawio
  - dev/mem
  - physical memory
  - ioperm iopl
  - container escape
---

# CAP_SYS_RAWIO

`CAP_SYS_RAWIO` authorises raw hardware I/O: the `ioperm(2)` and `iopl(2)` syscalls for x86 port access, and open/read/write on the raw memory devices `/dev/mem`, `/dev/kmem`, and `/dev/port` where they exist and are not blocked by `CONFIG_STRICT_DEVMEM`. Because physical memory and I/O ports are a property of the single host machine, not of any namespace, writing to them reaches kernel memory directly and ignores container boundaries.

Confirm the capability and that a usable device node is present:

```bash
capsh --print | grep -o cap_sys_rawio
grep CapEff /proc/self/status          # bit 17 set
ls -l /dev/mem /dev/port 2>/dev/null   # need a raw memory device exposed in the container
```

## Route: write kernel memory via /dev/mem

With `/dev/mem` readable and writable, the attacker maps physical pages, scans for a known kernel structure, and patches it. A common objective is to locate the credentials of a process and overwrite its UID/GID fields to 0, or to flip a security flag such as an LSM enforcement toggle.

```c
int fd = open("/dev/mem", O_RDWR);
// mmap a window of physical memory and scan for a signature
void *p = mmap(0, LEN, PROT_READ|PROT_WRITE, MAP_SHARED, fd, phys_off);
// locate task_struct->cred, overwrite uid/gid/euid/egid with 0
```

Translating a kernel virtual address to a physical offset for `/dev/mem` requires leaking the kernel base (for example from `/proc/kallsyms` if readable, or a known offset for the running kernel). Where `/dev/mem` is restricted to the first 1 MiB by `STRICT_DEVMEM`, the port and `iopl` paths remain for hardware that maps control registers into I/O space.

## Route: I/O port access

`iopl(3)` raises the process I/O privilege level so that `in`/`out` instructions on any port succeed, and `ioperm` enables specific ports. This reaches device controllers directly and, on suitable hardware, DMA-capable peripherals, which can be steered to read or write arbitrary physical memory.

## Exploitation notes

- Modern kernels build with `CONFIG_STRICT_DEVMEM` and often `CONFIG_IO_STRICT_DEVMEM`, which confine `/dev/mem` to device ranges and block RAM access; check `zcat /proc/config.gz | grep DEVMEM` or `/boot/config-$(uname -r)`. Where RAM is blocked this capability loses most of its escape value.
- This route is hardware- and offset-sensitive and is best reserved for cases where cleaner capabilities ([CAP_SYS_MODULE](cap-sys-module.md), [CAP_SYS_ADMIN](cap-sys-admin.md)) are absent but a raw memory device is exposed.
- Exposed raw memory devices usually mean a `--privileged` container or an explicit `--device=/dev/mem`; see [Kernel memory devices](device-access/kernel-memory-devices.md).

## References

- [man 2 iopl](https://man7.org/linux/man-pages/man2/iopl.2.html)
- [man 4 mem](https://man7.org/linux/man-pages/man4/mem.4.html)
- [HackTricks: CAP_SYS_RAWIO](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_sys_rawio)
