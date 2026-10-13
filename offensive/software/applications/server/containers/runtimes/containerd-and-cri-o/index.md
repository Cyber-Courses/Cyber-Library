---
title: "containerd and CRI-O: attacking the Kubernetes node runtimes"
order: 3
description: "containerd and CRI-O are the runtimes that actually run containers under Kubernetes. Their control sockets, if reachable, give node-level container control equivalent to the Docker socket, their clients (ctr, crictl) enumerate everything on the node, and the image-pull credentials they hold unlock private registries."
keywords:
  - containerd
  - cri-o
  - crictl
  - ctr
  - kubernetes node
---

# containerd and CRI-O

Under Kubernetes, the container that runs a pod is created not by Docker but by `containerd` or `CRI-O`, invoked by the kubelet over the Container Runtime Interface. Their offensive surface mirrors Docker's but is node-scoped: a reachable control socket is root-equivalent on the node, their clients enumerate every container, image, and task on the node, and the credentials they cache for pulling images unlock private registries.

Detect which runtime a node uses and find its socket:

```bash
ls -l /run/containerd/containerd.sock /run/crio/crio.sock /var/run/crio/crio.sock 2>/dev/null
crictl --runtime-endpoint unix:///run/containerd/containerd.sock version 2>/dev/null
ps -ef | grep -E 'containerd|crio' | grep -v grep | head
```

## Subtopics

- **[containerd socket](containerd-socket.md)**: node-level container control through the containerd socket.
- **[CRI-O socket](cri-o-socket.md)**: the same through the CRI-O socket.
- **[crictl and ctr enumeration](crictl-and-ctr-enumeration.md)**: mapping the node with the runtime clients.
- **[Image pull secret theft](image-pull-secret-theft.md)**: stealing the registry credentials the runtime uses.

## References

- [containerd documentation](https://github.com/containerd/containerd/tree/main/docs)
- [CRI-O documentation](https://github.com/cri-o/cri-o/tree/main/docs)
- [Kubernetes: container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
