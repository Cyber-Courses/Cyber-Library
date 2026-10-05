---
title: "containerd and CRI-O: attacking the low-level OCI runtimes"
description: "Attacking the low-level OCI runtimes that sit under Docker and Kubernetes: the containerd and CRI-O control sockets driven by ctr, nerdctl, and crictl, which create and run containers as root, and the image pull secrets these runtimes store on the node."
keywords:
  - containerd
  - CRI-O
  - ctr crictl
  - container runtime socket
  - kubernetes node
---

# containerd and CRI-O

Under Docker and Kubernetes sit the low-level runtimes that actually run containers: containerd (with runc) and CRI-O. On a Kubernetes node these are the engine, and their control sockets are root-equivalent just like the Docker daemon. Reaching one from a node foothold creates privileged containers and reads everything the runtime stores, including image pull credentials.

## Subtopics

- **[containerd socket](containerd-socket.md)**: the containerd control socket via ctr and nerdctl.
- **[CRI-O socket](cri-o-socket.md)**: the CRI runtime socket via crictl.
- **[crictl and ctr enumeration](crictl-and-ctr-enumeration.md)**: inventorying containers and images.
- **[Image pull secret theft](image-pull-secret-theft.md)**: recovering registry credentials from the runtime.

## References

- [containerd documentation](https://containerd.io/docs/)
- [CRI-O documentation](https://github.com/cri-o/cri-o/tree/main/docs)
