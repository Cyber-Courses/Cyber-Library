---
title: "Container escape: breaking out of a container to the host"
description: "Runtime-agnostic container breakout: escaping to the host through over-permissive configuration (the privileged flag, dangerous capabilities, device access, weak confinement), dangerous bind mounts, shared host namespaces, runtime and kernel exploits, and escapes from sandboxed runtimes."
keywords:
  - container escape
  - container breakout
  - privileged container
  - host namespace
  - container runtime exploit
---

# Container escape

A container escape turns code execution inside a container into code execution on the host. It almost never depends on the engine: the same primitives work from Docker, Podman, containerd, and a Kubernetes pod, because they all lean on the same kernel features. What differs is only how the dangerous setting was handed to the container, which is why the runtime and orchestration layers link here rather than re-document the breakout.

## Triage first

Before trying any route, fingerprint what the container was given; the output decides which subtopic applies and saves wasted attempts:

```bash
# Capabilities, seccomp, and LSM confinement
grep -E 'CapEff|Seccomp' /proc/self/status
capsh --decode=$(grep CapEff /proc/self/status | awk '{print $2}')
cat /proc/self/attr/current 2>/dev/null         # AppArmor profile, or "unconfined"

# Is container root real host root?
cat /proc/self/uid_map                           # "0 0 4294967295" => host root

# Host paths, sockets, and sensitive kernel files mounted in
findmnt -o TARGET,SOURCE,OPTIONS | grep -vE 'overlay|proc|sysfs|tmpfs|cgroup'
ls -l /var/run/docker.sock /run/containerd/containerd.sock 2>/dev/null
for f in /proc/sys/kernel/core_pattern /proc/sys/kernel/modprobe /sys/kernel/uevent_helper; do
  [ -w "$f" ] && echo "writable: $f"; done

# Shared host namespaces
ls /proc | grep -qE '^[0-9]+$' && readlink /proc/1/exe   # host init => shared PID ns
ip addr | grep -q 'docker0\|host' && ss -tlnp 2>/dev/null | head

# Devices and runtime versions (for the exploit routes)
ls -l /dev | grep -E 'sd|nvme|mem' ; runc --version 2>/dev/null; uname -r
```

Read it in order: a full `CapEff` with `Seccomp: 0` is `--privileged`; a single added capability points to one [Capability abuse](privileged-configuration/capability-abuse/index.md) page; a host bind mount or socket is a [Sensitive mount](sensitive-mounts/index.md); shared namespaces route to [Shared host namespaces](shared-host-namespaces/index.md); and a locked-down container with none of these leaves only [Runtime and kernel exploits](runtime-and-kernel-exploits/index.md). A mapped `uid_map` warns that capability routes may yield only a remapped identity, not host root.

The primitives group by what makes the escape possible.

## Subtopics

- **[Privileged configuration](privileged-configuration/index.md)**: the container was given too much, from the all-in-one privileged flag to a single dangerous capability, raw device access, or a disabled seccomp or AppArmor profile.
- **[Sensitive mounts](sensitive-mounts/index.md)**: a host path, a runtime control socket, or host procfs and sysfs was bind-mounted into the container.
- **[Shared host namespaces](shared-host-namespaces/index.md)**: the container shares the host PID, network, IPC, or user namespace.
- **[Runtime and kernel exploits](runtime-and-kernel-exploits/index.md)**: a code-execution flaw in the OCI runtime or shim, or in the shared host kernel.
- **[Sandboxed runtime escapes](sandboxed-runtime-escapes/index.md)**: breaking out of a sandbox that interposes on the kernel, such as gVisor or a microVM runtime.

## References

- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [NCC Group: Understanding and Hardening Linux Containers](https://research.nccgroup.com/2016/04/13/understanding-and-hardening-linux-containers/)
- [man 7 namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)
