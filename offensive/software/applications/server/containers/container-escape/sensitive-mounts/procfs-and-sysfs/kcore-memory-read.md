---
title: "kcore memory read: dumping host kernel memory through /proc/kcore"
order: 5
description: "/proc/kcore presents the live kernel's memory as an ELF core file. A container that can read the host's /proc/kcore reads physical and kernel virtual memory, carving out secrets, keys, and credential structures. It is a powerful read primitive that bypasses namespace isolation because kernel memory is a single host-wide resource."
keywords:
  - proc kcore
  - kernel memory
  - memory carving
  - information disclosure
  - container escape
---

# kcore memory read

`/proc/kcore` is a virtual ELF core file that maps the running kernel's memory, including physical RAM and kernel virtual address space. Anything in kernel memory, and much of what is in RAM, can be read through it. Because kernel memory is a single host-wide resource not divided by namespaces, reading the host's `/proc/kcore` from inside a container discloses host secrets regardless of container isolation. It is a read-only primitive, but a far-reaching one.

Confirm access and size:

```bash
ls -l /proc/kcore                              # present and readable
head -c 4 /proc/kcore | xxd                    # ELF magic (7f 45 4c 46) confirms the core format
```

## Carving memory

The ELF program headers in `/proc/kcore` describe which virtual ranges map to which offsets. Tools parse these to read specific kernel addresses; a blunt approach scans readable segments for recognisable secrets.

```bash
# Scan readable memory for key material and credentials
strings -n 12 /proc/kcore 2>/dev/null | grep -iE 'BEGIN (RSA|OPENSSH|EC) PRIVATE KEY' | head
strings /proc/kcore 2>/dev/null | grep -iE 'password|secret|token' | head
# Targeted: use a kernel symbol address from /proc/kallsyms, translate via the
# kcore ELF program headers, then read that offset
grep ' D ' /proc/kallsyms | grep -i init_task  # example symbol to anchor a read
```

With a kernel symbol table (`/proc/kallsyms` if readable) and the kcore program headers, specific structures such as process credentials or key rings can be located and read precisely, rather than relying on string scanning.

## Exploitation notes

- This gives read, not write; it cannot by itself execute code. Its value is extracting secrets (private keys, cloud and service tokens, `/etc/shadow` cached in memory, encryption keys) that unlock the host or other systems.
- Precision reads need `/proc/kallsyms` with real addresses; if `kptr_restrict` zeroes the symbols, fall back to signature scanning with `strings` and pattern matches.
- Access usually implies a privileged container or an explicit host `/proc` mount; a namespaced container does not expose a usable host `kcore`. For write-capable kernel memory access see [Kernel memory devices](../../privileged-configuration/device-access/kernel-memory-devices.md).

## Tools

- [volatility3 (memory analysis, can parse kcore-style sources)](https://github.com/volatilityfoundation/volatility3)

## References

- [man 5 proc: /proc/kcore](https://man7.org/linux/man-pages/man5/proc.5.html)
- [HackTricks: /proc/kcore](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts#proc-kcore)
