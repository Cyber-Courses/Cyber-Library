---
title: "Base image poisoning: tampering with upstream base images"
description: "Poisoning the base images that downstream builds depend on, by typosquatting a public image name, hijacking an unpinned mutable tag, or compromising a shared internal base image, so every build FROM it inherits the attacker's code."
keywords:
  - base image poisoning
  - typosquatting
  - mutable tag
  - FROM image
  - supply chain
---

# Base image poisoning

A `FROM` line is a trust decision. Builds that pull an unpinned base image by a mutable tag inherit whatever that tag points to at build time. An attacker poisons the base: typosquatting a public name close to a popular one, hijacking an abandoned namespace, or compromising a shared internal base image that many teams build on.

```dockerfile
# Unpinned base: resolves to whatever :latest points to at build time
FROM company/base:latest
# Pinned by digest resists poisoning
FROM company/base@sha256:<digest>
```

## Exploitation notes

- The leverage is fan-out: one poisoned shared base image compromises every downstream image and deployment that rebuilds.
- Typosquats target common names (one character off, a different registry namespace) and rely on a typo in a Dockerfile or CI config.
- Pinning by digest defeats tag mutation, so unpinned `FROM` lines are the targets to look for.

## References

- [SLSA: supply chain threats](https://slsa.dev/spec/v1.0/threats)
- [OCI image specification](https://github.com/opencontainers/image-spec)
