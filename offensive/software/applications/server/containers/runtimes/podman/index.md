---
title: "Podman: attacking the daemonless container engine"
description: "Attacking Podman, the daemonless and rootless-capable container engine: reaching its optional API service socket, abusing the rootless user-namespace model and its limits, and using the companion tools skopeo and buildah with the user's stored registry credentials."
keywords:
  - podman
  - rootless containers
  - podman API socket
  - skopeo buildah
  - container runtime
---

# Podman

Podman runs containers without a central daemon: each `podman` invocation forks the runtime directly, and it can run fully rootless under a user's own account. That changes the attack surface from Docker's: there is no always-on root daemon, but there is an optional API service socket, a rootless model with its own escape limits, and the same OCI image and runtime primitives underneath.

## Subtopics

- **[API service socket](api-service-socket.md)**: the optional Podman REST API.
- **[systemd socket activation](systemd-socket-activation.md)**: the socket exposed through systemd.
- **[Rootless model](rootless-model.md)**: the rootless user-namespace mapping and its limits.
- **[skopeo and buildah](skopeo-and-buildah.md)**: the companion image tools and their credentials.

## References

- [Podman documentation](https://docs.podman.io/)
- [Podman REST API](https://docs.podman.io/en/latest/markdown/podman-system-service.1.html)
