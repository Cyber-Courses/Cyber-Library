---
title: "Build-time: attacking the Docker image build"
description: "Attacking the image build rather than a running container: recovering secrets passed as build arguments or baked into intermediate layers, and executing code on the build host through BuildKit flaws or a malicious Dockerfile and build context."
keywords:
  - docker build
  - buildkit
  - build-arg secret
  - build host RCE
  - supply chain
---

# Build-time

The build is a privileged step that runs commands and pulls dependencies, often in CI on a host that also builds other projects. Two things go wrong there: secrets handed to the build persist in the image, and the build itself can be made to run attacker code on the build host.

## Subtopics

- **[Build-arg secret leak](build-arg-secret-leak.md)**: secrets in build args and intermediate layers.
- **[BuildKit RCE](buildkit-rce.md)**: code execution on the build host.

## References

- [Docker build secrets](https://docs.docker.com/build/building/secrets/)
- [BuildKit](https://github.com/moby/buildkit)
