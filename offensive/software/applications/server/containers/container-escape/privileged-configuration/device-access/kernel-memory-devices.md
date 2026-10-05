---
title: "Kernel memory devices: patching the host kernel through /dev/mem"
description: "Escaping a container that can access kernel and physical memory devices such as /dev/mem and /dev/kmem by reading kernel memory for secrets and addresses or writing it to patch the running host kernel."
keywords:
  - /dev/mem
  - /dev/kmem
  - kernel memory
  - container escape
  - physical memory
---

# Kernel memory devices

`/dev/mem` and `/dev/kmem` expose physical and kernel virtual memory. Where they are present in the container (privileged, or granted) and readable, they allow dumping kernel memory for secrets and addresses; where writable, they allow patching the live kernel.

```bash
ls -l /dev/mem 2>/dev/null
dd if=/dev/mem bs=1M count=32 2>/dev/null | strings -n 12 | grep -i 'key\|token' | head
```

## Exploitation notes

- Modern kernels restrict `/dev/mem` to the first megabyte unless `CONFIG_STRICT_DEVMEM` is off, so broad reads need a permissive host configuration or `CAP_SYS_RAWIO`.
- Reading for KASLR and secrets is portable; writing kernel structures (for example a credential check) is version-specific and risky.
- This pairs with [CAP_SYS_RAWIO](../capability-abuse/cap-sys-rawio.md).

## References

- [man 4 mem](https://man7.org/linux/man-pages/man4/mem.4.html)
- [Kernel: STRICT_DEVMEM](https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html)
