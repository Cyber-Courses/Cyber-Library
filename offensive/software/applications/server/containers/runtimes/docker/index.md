---
title: "Docker: attacking the daemon, images, and build"
description: "Attacking the Docker engine: reaching its root-equivalent daemon API over an exposed port or weak TLS, looting and poisoning images and registries, and leaking secrets or executing code during the image build. The container-escape primitives Docker shares with every runtime live separately."
keywords:
  - docker
  - docker daemon API
  - docker registry
  - docker build
  - container security
---

# Docker

Docker splits into a client and a root daemon that does the work. Offensive interest is in the daemon's API, which is a full root-equivalent control plane, and in the images, registries, and builds that feed it. Escaping a container you start through Docker uses the shared primitives in [Container escape](../../container-escape/index.md); this area is everything that is specifically Docker.

## Subtopics

- **[Exposed daemon API](exposed-daemon-api/index.md)**: reaching the daemon over the network or weak TLS.
- **[Images and registries](images-and-registries/index.md)**: looting, poisoning, and enumerating images and registries.
- **[Build-time](build-time/index.md)**: secrets and code execution during the image build.

## References

- [Docker engine security](https://docs.docker.com/engine/security/)
- [Docker Engine API](https://docs.docker.com/engine/api/)
