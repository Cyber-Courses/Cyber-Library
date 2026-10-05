---
title: "Images and registries: attacking the container supply chain"
description: "Container images are distributed through registries and assembled from base images, both of which are supply-chain attack surfaces. An attacker enumerates and pulls from exposed or unauthenticated registries, extracts secrets baked into image layers, and poisons or backdoors base images so that every downstream build runs attacker code."
keywords:
  - container registry
  - docker image
  - supply chain
  - image layers
  - base image
---

# Images and registries

An image is a stack of filesystem layers plus a configuration, and a registry is the server that stores and serves them. Both are supply-chain targets. A registry that is reachable without authentication, or with weak credentials, lets an attacker pull private images (and the secrets inside them) and sometimes push malicious ones. The images themselves leak secrets that were added during the build and later thought removed, because every layer is retained. And a base image an attacker can influence poisons every image built on top of it.

Find registries and inspect images:

```bash
# registry API v2 is unauthenticated to probe by default
curl -s http://<registry>:5000/v2/_catalog
curl -s http://<registry>:5000/v2/<repo>/tags/list
# pull and inspect an image's build history for secrets
docker pull <registry>:5000/<repo>:<tag>
docker history --no-trunc <registry>:5000/<repo>:<tag>
```

## Subtopics

- **[Registry enumeration](registry-enumeration.md)**: listing repositories, tags, and manifests.
- **[Registry access](registry-access.md)**: pulling and pushing with found or weak credentials.
- **[Unauthenticated registry access](unauthenticated-registry-access.md)**: an open registry with no auth.
- **[Secrets in image layers](secrets-in-image-layers.md)**: extracting credentials baked into layers.
- **[Base image poisoning](base-image-poisoning.md)**: compromising an upstream image to reach downstreams.
- **[Image backdooring](image-backdooring.md)**: modifying an image to run attacker code on launch.

## References

- [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec/blob/main/spec.md)
- [Docker Registry HTTP API V2](https://distribution.github.io/distribution/spec/api/)
- [HackTricks: 5000 Docker registry](https://book.hacktricks.xyz/network-services-pentesting/5000-pentesting-docker-registry)
