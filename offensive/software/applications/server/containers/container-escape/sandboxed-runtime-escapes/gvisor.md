---
title: "gVisor: escaping the userspace kernel sandbox"
description: "Escaping gVisor, which runs a userspace kernel (the Sentry) that emulates Linux syscalls for the container, by finding a flaw in the Sentry's syscall emulation or in the narrow host interface it uses, reaching the real host kernel behind the sandbox."
keywords:
  - gVisor
  - Sentry
  - userspace kernel
  - sandbox escape
  - syscall emulation
---

# gVisor

gVisor interposes a userspace kernel, the Sentry, between the container and the host. The container's syscalls hit the Sentry, not the host kernel, so most container escapes and kernel exploits simply do not reach the host. An escape instead needs a bug in the Sentry's own syscall emulation or in the restricted host surface (the Gofer and the small set of host syscalls the Sentry itself makes).

```bash
# Confirm the sandbox: gVisor reports a distinctive kernel identity
uname -a            # gVisor-specific version string
dmesg 2>/dev/null | head
```

## Exploitation notes

- The host kernel surface is deliberately tiny, so the realistic targets are logic flaws in Sentry syscall handling or the file-proxy (Gofer) interface.
- Standard primitives like a privileged flag or a mounted socket do not translate: there is no host kernel behind the syscall to abuse.
- Fingerprint gVisor early, because it changes which techniques are even applicable.

## References

- [gVisor security model](https://gvisor.dev/docs/architecture_guide/security/)
- [gVisor architecture guide](https://gvisor.dev/docs/architecture_guide/)
