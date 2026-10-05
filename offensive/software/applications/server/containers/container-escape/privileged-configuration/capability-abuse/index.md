---
title: "Capability abuse: escaping through a single dangerous Linux capability"
description: "Container escape through one over-granted Linux capability. Each capability exposes a specific host-reaching primitive: mounting filesystems, loading kernel modules, reading arbitrary host files, injecting into host processes, or reaching physical memory."
keywords:
  - linux capabilities
  - CAP_SYS_ADMIN
  - CAP_SYS_MODULE
  - CAP_DAC_READ_SEARCH
  - container escape
---

# Capability abuse

Docker and Kubernetes drop most capabilities by default, but workloads routinely add one or two back for convenience, and each dangerous capability is its own escape. Read the effective set, then pick the matching technique.

```bash
capsh --print | sed -n 's/^Current: //p'
grep CapEff /proc/self/status        # decode with: capsh --decode=<hex>
```

## Subtopics

- **[CAP_SYS_ADMIN](cap-sys-admin.md)**: mount, cgroups, and namespace operations.
- **[CAP_SYS_PTRACE](cap-sys-ptrace.md)**: inject into host processes.
- **[CAP_SYS_MODULE](cap-sys-module.md)**: load a kernel module.
- **[CAP_DAC_READ_SEARCH](cap-dac-read-search.md)**: read any host file.
- **[CAP_DAC_OVERRIDE](cap-dac-override.md)**: write host files past permissions.
- **[CAP_SYS_RAWIO](cap-sys-rawio.md)**: raw I/O and physical memory.
- **[CAP_NET_RAW](cap-net-raw.md)**: craft and sniff raw packets.
- **[CAP_BPF](cap-bpf.md)**: load eBPF programs.

## References

- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [HackTricks: Linux capabilities](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/linux-capabilities.html)
