---
title: "CAP_BPF: reading and writing kernel memory through BPF programs"
description: "CAP_BPF allows loading BPF programs and creating maps. On its own, or combined with CAP_PERFMON or CAP_SYS_ADMIN, it provides a path to read arbitrary kernel memory through bpf_probe_read and, where tracing program types and helpers are reachable, to overwrite kernel data, since BPF programs execute in kernel context shared across all namespaces."
keywords:
  - cap_bpf
  - bpf syscall
  - bpf_probe_read
  - kernel memory
  - container escape
---

# CAP_BPF

`CAP_BPF` was split out of `CAP_SYS_ADMIN` to allow loading BPF programs and creating BPF maps without the full admin capability. BPF programs run inside the kernel after passing the verifier, and they share the single host kernel across every namespace. With `CAP_BPF`, and depending on which program types and helpers the kernel allows, an attacker can read kernel memory and, with tracing programs plus helpers such as `bpf_probe_write_user` or map-based primitives, influence kernel and user memory, escaping container isolation.

Confirm the capability and the reachable program types:

```bash
capsh --print | grep -o -E 'cap_bpf|cap_perfmon|cap_sys_admin'
grep CapEff /proc/self/status          # cap_bpf is bit 39
bpftool feature probe 2>/dev/null | grep -i 'program_type .* available' | head
```

## Route: read kernel memory

A tracing BPF program (for example `BPF_PROG_TYPE_KPROBE` or `BPF_PROG_TYPE_TRACEPOINT`, which usually also need `CAP_PERFMON`) can call `bpf_probe_read_kernel` to copy arbitrary kernel addresses into a map the attacker then reads from user space. This leaks credentials, pointers for defeating KASLR, and the contents of kernel structures.

```c
// sketch of a kprobe program body (loaded via bpf() BPF_PROG_LOAD)
// reads N bytes at a target kernel address into a perf/array map
bpf_probe_read_kernel(&out, sizeof(out), (void *)target_kaddr);
bpf_map_update_elem(&results, &key, &out, BPF_ANY);
```

```bash
# user side: pin the map, poll it for leaked kernel data
bpftool map dump name results
```

## Route: write primitives

Where `bpf_probe_write_user` is available to the loaded program type, a tracing program that fires in the context of a target task can overwrite that task's user memory. Combined with a helper that reaches a credentials structure, this is escalated to overwriting UID fields or disabling a check. The exact reachable helpers depend on kernel version and the capability combination; `CAP_BPF` with `CAP_PERFMON` unlocks the tracing program types that make these helpers usable.

## Exploitation notes

- The BPF verifier and the per-program-type helper allowlists are the real gate. `CAP_BPF` alone restricts you to a subset; pairing with `CAP_PERFMON` (tracing) or `CAP_SYS_ADMIN` greatly widens what loads. Probe with `bpftool feature probe`.
- `kernel.unprivileged_bpf_disabled` does not apply here because you hold the capability, but `CONFIG_BPF_UNPRIV_DEFAULT_OFF` and a locked-down `bpf()` surface can still block specific program types.
- Historically several BPF verifier bugs allowed arbitrary kernel read/write from loadable programs; where the host kernel is unpatched, a verifier-bypass program is a direct escape rather than only a leak.

## Tools

- [bpftool (kernel BPF inspection and loading)](https://github.com/libbpf/bpftool)

## References

- [man 2 bpf](https://man7.org/linux/man-pages/man2/bpf.2.html)
- [Kernel docs: BPF design and capabilities](https://docs.kernel.org/bpf/index.html)
- [HackTricks: CAP_BPF](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)
