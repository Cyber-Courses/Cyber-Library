---
title: "Docker: attacking the engine, its API, images, and builds"
description: "Offensive surface of the Docker engine: a daemon API that is root-equivalent and often exposed over the network, images and registries that leak secrets and carry supply-chain risk, and a build process that leaks credentials and has its own vulnerabilities. Container escape itself is runtime-agnostic and documented separately."
keywords:
  - docker
  - docker daemon
  - docker registry
  - docker build
  - container security
---

# Docker

Docker is the most widely deployed container engine, and its offensive surface is the engine and its supporting services rather than the container boundary, which is a kernel property shared with every other runtime. The daemon API is root-equivalent and frequently reachable over the network; images and registries hold secrets and are a supply-chain target; and the build process is an execution environment that leaks credentials. Escaping a Docker container to the host uses the same primitives as any runtime and is covered under container escape.

```bash
docker version 2>/dev/null; docker info 2>/dev/null | grep -iE 'rootless|security'
ss -tlnp 2>/dev/null | grep -E '2375|2376'          # daemon exposed over TCP
ls -l /var/run/docker.sock 2>/dev/null               # local socket reachable
```

## Subtopics

- **[Exposed daemon API](exposed-daemon-api/index.md)**: controlling the root-equivalent daemon over the network.
- **[Images and registries](images-and-registries/index.md)**: secrets in images and registry supply-chain attacks.
- **[Build-time attacks](build-time/index.md)**: leaking secrets and exploiting the build process.

## References

- [Docker Engine API reference](https://docs.docker.com/reference/api/engine/)
- [Docker: security](https://docs.docker.com/engine/security/)
- [HackTricks: Docker security](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security)
