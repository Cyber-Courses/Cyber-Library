---
title: "Linux: attacking the Linux host platform"
description: "Offensive techniques against a Linux host: local privilege escalation through SUID and SGID binaries, sudo rules, Linux capabilities, cron, writable paths, and PATH abuse, credential and secret access, abuse of container and namespace boundaries, and kernel exploitation."
keywords:
  - Linux privilege escalation
  - SUID
  - sudo
  - Linux capabilities
  - kernel exploitation
---

# Linux

Linux is the dominant server and infrastructure platform, and its local attack surface flows from the Unix privilege model: a single `root` superuser, file-permission and ownership rules, and a handful of mechanisms that deliberately grant elevated execution. Once code runs as an unprivileged user, the work is to find one of those mechanisms misconfigured and ride it to `root`, then harvest the secrets the host holds.

## The local surface

- **Privilege escalation**: SUID/SGID binaries (and the GTFOBins that abuse them), permissive `sudo` rules, overly broad file **capabilities**, writable `cron` and systemd units, `PATH` and library (`LD_PRELOAD`/`LD_LIBRARY_PATH`) hijacking, and writable sensitive files (`/etc/passwd`, `/etc/shadow`, sudoers).
- **Credential and secret access**: history files, world-readable configs and keys, SSH keys and agents, service credentials, and in-memory secrets.
- **Container and namespace boundaries**: escaping a container to its host is its own deep area under [Containers](../../applications/server/containers/index.md); the namespace, cgroup, and capability primitives it abuses are the same ones that gate privilege here.
- **Kernel**: local kernel exploitation (the classic overwrite-and-escalate primitives) for the final step to `root`.

## Seams

Container escape is covered under [Containers](../../applications/server/containers/index.md), and hypervisor guest-to-host escape under [Virtualization](../../applications/server/virtualization/index.md); both are cross-referenced. This area is the Linux host itself.

## References

- [GTFOBins](https://gtfobins.github.io/)
- [MITRE ATT&CK: privilege escalation](https://attack.mitre.org/tactics/TA0004/)
