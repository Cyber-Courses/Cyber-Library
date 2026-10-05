---
title: "Container runtimes: attacking the engines and their control planes"
description: "Attacking the container engines themselves rather than escaping a container: the daemon and API sockets each exposes, which are root-equivalent control planes, and the image and registry supply chain that feeds them, across Docker, Podman, and the low-level containerd and CRI-O runtimes."
keywords:
  - container runtime
  - docker daemon
  - podman
  - containerd
  - CRI-O
---

# Runtimes

A container engine is a privileged service that builds images, pulls them from registries, and starts containers as root. Attacking the engine is distinct from escaping a container: the targets are the control plane it exposes (a daemon API or a control socket, almost always root-equivalent) and the image and registry supply chain behind it. The actual host breakout, once you can start a container, is the same everywhere and lives under [Container escape](../container-escape/index.md).

## Subtopics

- **[Docker](docker/index.md)**: the daemon API and the image and registry and build supply chain.
- **[Podman](podman/index.md)**: the daemonless, rootless-capable engine and its API socket.
- **[containerd and CRI-O](containerd-and-cri-o/index.md)**: the low-level OCI runtimes under Docker and Kubernetes.

## References

- [Docker engine security](https://docs.docker.com/engine/security/)
- [OCI distribution specification](https://github.com/opencontainers/distribution-spec)
