---
title: "Exposed daemon API: controlling Docker over the network"
description: "The Docker daemon exposes a full control API. When it is bound to a TCP port, reachable without authentication on 2375 or protected only by weak TLS on 2376, anyone who can reach it has root-equivalent control of the host: they enumerate the environment and then launch a privileged container that mounts the host filesystem."
keywords:
  - docker daemon
  - docker api
  - port 2375
  - remote docker
  - host takeover
---

# Exposed daemon API

The Docker daemon's API is its entire control surface, and it is root-equivalent by design: anyone who can call it can start a container that mounts the host and runs as root. Locally the API lives on a Unix socket, but operators frequently expose it over TCP for remote management, on 2375 (plain HTTP, no authentication) or 2376 (TLS). An exposed 2375, or a 2376 with weak or optional TLS, hands full host control to any client that can reach the port, with no exploit required.

Find and probe an exposed daemon:

```bash
# scan for the Docker API ports
nmap -p 2375,2376 --open <target-range>
# a single unauthenticated request confirms control
curl -s http://<target>:2375/version
curl -s http://<target>:2375/info | head
```

A `200` with daemon version and info from 2375 means unauthenticated control.

## Subtopics

- **[Unauthenticated daemon access](unauthenticated-daemon-access.md)**: an open 2375 with no authentication.
- **[Weak TLS on 2376](weak-tls-on-2376.md)**: TLS present but misconfigured or optional.
- **[API enumeration](api-enumeration.md)**: mapping containers, images, secrets, and networks through the API.
- **[Host takeover via privileged run](host-takeover-via-privileged-run.md)**: launching a container that owns the host.

## References

- [Docker Engine API reference](https://docs.docker.com/reference/api/engine/)
- [Docker: protect the daemon socket](https://docs.docker.com/engine/security/protect-access/)
- [HackTricks: 2375 Docker](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
