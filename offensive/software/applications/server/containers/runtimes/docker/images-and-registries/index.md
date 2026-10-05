---
title: "Images and registries: attacking the Docker image supply chain"
description: "Attacking the Docker image and registry supply chain: enumerating and accessing registries, pulling private images, recovering secrets baked into image layers and history, and poisoning base images or planting backdoored images that run when deployed."
keywords:
  - docker registry
  - image supply chain
  - image secrets
  - registry enumeration
  - image backdoor
---

# Images and registries

Images are code and configuration shipped as layers, and registries are where they live. The supply chain is an attack surface in both directions: pull to read private images and the secrets inside them, and push to plant a backdoored or poisoned image that executes when someone deploys it.

## Subtopics

- **[Registry enumeration](registry-enumeration.md)**: discovering repositories and tags.
- **[Registry access](registry-access.md)**: authenticating to and pulling from a registry.
- **[Unauthenticated registry access](unauthenticated-registry-access.md)**: anonymous pull and push.
- **[Secrets in image layers](secrets-in-image-layers.md)**: credentials baked into layers and history.
- **[Image backdooring](image-backdooring.md)**: planting a malicious image.
- **[Base image poisoning](base-image-poisoning.md)**: tampering with upstream base images.

## References

- [OCI distribution specification](https://github.com/opencontainers/distribution-spec)
- [Docker registry HTTP API v2](https://distribution.github.io/distribution/spec/api/)
