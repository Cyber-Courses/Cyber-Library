---
title: "Runtimes: attacking the engines that build and run containers"
description: "The container runtimes themselves are an attack surface distinct from the container boundary: the Docker engine and its network API, the Kubernetes node runtimes containerd and CRI-O with their control sockets, and Podman's rootless and socket-activated model. Each exposes control planes, credentials, and image handling that lead to host or node compromise."
keywords:
  - container runtime
  - docker
  - containerd
  - cri-o
  - podman
---

# Runtimes

A container runtime does far more than start processes: it exposes a control API, pulls and stores images with cached registry credentials, builds images, and runs as root (or near-root) on the host. That machinery is an attack surface in its own right, separate from escaping the container boundary, which is a shared-kernel property documented under container escape. The runtimes differ in how they expose this surface: Docker through a root-equivalent daemon API, containerd and CRI-O through node control sockets under Kubernetes, and Podman through a deliberately rootless, socket-activated design.

```bash
# identify the runtimes present and their control surfaces
docker info 2>/dev/null | grep -iE 'rootless|server version'
ls -l /run/containerd/containerd.sock /run/crio/crio.sock 2>/dev/null
ls -l /run/podman/podman.sock /run/user/*/podman/podman.sock 2>/dev/null
ps -ef | grep -E 'dockerd|containerd|crio|podman' | grep -v grep
```

## Subtopics

- **[Docker](docker/index.md)**: the engine API, images and registries, and the build process.
- **[containerd and CRI-O](containerd-and-cri-o/index.md)**: the Kubernetes node runtimes and their sockets.
- **[Podman](podman/index.md)**: the rootless, daemonless, socket-activated model.

## References

- [Docker: security](https://docs.docker.com/engine/security/)
- [Kubernetes: container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
- [Podman documentation](https://docs.podman.io/)
