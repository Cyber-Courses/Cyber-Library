---
title: "Podman: attacking a rootless, daemonless runtime"
order: 2
description: "Podman runs containers without a central daemon and defaults to rootless operation, which changes its attack surface. The optional API socket still grants container control, the rootless user-namespace model determines what an escape actually yields, and the companion tools skopeo and buildah handle images and builds with their own credential exposure."
keywords:
  - podman
  - rootless containers
  - podman socket
  - skopeo
  - buildah
---

# Podman

Podman is a container engine designed to run without a central privileged daemon and, by default, without root. That reshapes its offensive surface. There is no always-on root daemon to take over, but Podman can expose a Docker-compatible API socket that grants container control; its rootless model, built on user namespaces, decides whether a container escape lands as an unprivileged user or as root; and its companion tools, `skopeo` for image transport and `buildah` for builds, carry the same registry-credential and image-secret exposure as Docker.

```bash
podman info 2>/dev/null | grep -iE 'rootless|graphRoot|runRoot'
ls -l /run/podman/podman.sock /run/user/*/podman/podman.sock 2>/dev/null
cat /proc/self/uid_map                               # rootless => mapped range, not 0 0
```

## Subtopics

- **[Rootless model](rootless-model.md)**: how the user-namespace design constrains or enables escapes.
- **[API service socket](api-service-socket.md)**: the optional Docker-compatible control socket.
- **[systemd socket activation](systemd-socket-activation.md)**: on-demand API activation and its exposure.
- **[skopeo and buildah](skopeo-and-buildah.md)**: image transport and build tooling credential exposure.

## References

- [Podman documentation](https://docs.podman.io/)
- [Podman: rootless containers](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)
- [Rootless containers](https://rootlesscontaine.rs/)
