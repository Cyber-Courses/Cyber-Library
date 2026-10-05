---
title: "Capability abuse: single Linux capabilities that each break containment"
description: "How individual Linux capabilities granted to a container each provide a distinct route to the host: CAP_SYS_ADMIN for mounts and the cgroup release_agent, CAP_SYS_MODULE for kernel module loading, CAP_DAC_READ_SEARCH for arbitrary host file reads, CAP_SYS_PTRACE for host process injection, CAP_SYS_RAWIO and CAP_BPF for direct kernel memory access, and CAP_NET_RAW for network interception."
keywords:
  - linux capabilities
  - cap_sys_admin
  - cap_sys_module
  - container escape
  - capabilities decode
---

# Capability abuse

A container does not need the full privileged flag to be escapable. A single capability, added with `--cap-add` or a Kubernetes `securityContext.capabilities.add`, can be enough, because each capability maps to a kernel code path that was never meant to be reachable by a semi-trusted workload. The default Docker capability set is already a trimmed list; the dangerous escapes come from capabilities that are added on top, or from a base image run with `--cap-add=ALL`.

Decode exactly what you hold before choosing a page:

```bash
grep CapEff /proc/self/status          # hex bitmask of effective capabilities
capsh --decode=00000000a80425fb        # => the Docker default set
capsh --print                          # human-readable current/bounding set
getpcaps $$                            # same, via libcap
```

Match a bit in that mask to the capability name, then go to the matching page. The highest-value additions, roughly in order of how directly they escape:

## Subtopics

- **[CAP_SYS_ADMIN](cap-sys-admin.md)**: mount, pivot_root, and the cgroup release_agent escape.
- **[CAP_SYS_MODULE](cap-sys-module.md)**: load a kernel module that runs code on the host.
- **[CAP_DAC_READ_SEARCH](cap-dac-read-search.md)**: read any host file by brute-forcing inode handles.
- **[CAP_DAC_OVERRIDE](cap-dac-override.md)**: bypass file permission checks to write host files.
- **[CAP_SYS_PTRACE](cap-sys-ptrace.md)**: inject code into host processes when PID namespace is shared.
- **[CAP_SYS_RAWIO](cap-sys-rawio.md)**: raw port and physical-memory access to patch the kernel.
- **[CAP_BPF](cap-bpf.md)**: load BPF programs that read and write kernel memory.
- **[CAP_NET_RAW](cap-net-raw.md)**: raw sockets for sniffing and spoofing on the container network.

## References

- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [HackTricks: Linux capabilities](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)
- [Docker default capabilities (moby/oci defaults)](https://github.com/moby/moby/blob/master/oci/caps/defaults.go)
