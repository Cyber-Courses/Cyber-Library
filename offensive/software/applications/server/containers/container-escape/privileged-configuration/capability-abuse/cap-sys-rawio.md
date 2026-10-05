---
title: "CAP_SYS_RAWIO: reaching physical memory and I/O ports from a container"
description: "Escaping a container that holds CAP_SYS_RAWIO by using raw I/O port access and physical memory devices such as /dev/mem to read and write kernel memory, patching the running host kernel or recovering secrets."
keywords:
  - CAP_SYS_RAWIO
  - /dev/mem
  - physical memory
  - container escape
  - iopl
---

# CAP_SYS_RAWIO

`CAP_SYS_RAWIO` permits raw I/O: `iopl`/`ioperm` port access and reads and writes of physical memory devices like `/dev/mem` and `/dev/port` when they are present. That is a direct window into kernel memory, which can be read for secrets and KASLR, or written to patch kernel structures.

```bash
capsh --print | grep -q cap_sys_rawio && echo have
ls -l /dev/mem /dev/port 2>/dev/null
# Read physical memory for kernel structures / secrets
dd if=/dev/mem bs=1M count=16 2>/dev/null | strings | head
```

## Exploitation notes

- The capability is only as useful as the exposed devices: a privileged container has `/dev/mem`; otherwise it must be granted with `--device`.
- Writing `/dev/mem` to patch the kernel (for example disabling a credential check) is powerful but fragile across kernel versions; reads for a [kernel exploit](../../runtime-and-kernel-exploits/kernel-exploits/index.md) are more portable.
- Related device routes are under [Kernel memory devices](../device-access/kernel-memory-devices.md).

## References

- [man 4 mem](https://man7.org/linux/man-pages/man4/mem.4.html)
- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
