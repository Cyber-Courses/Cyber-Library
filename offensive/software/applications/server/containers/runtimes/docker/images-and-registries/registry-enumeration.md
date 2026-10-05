---
title: "Registry enumeration: discovering repositories and tags"
description: "Enumerating a container registry through the v2 API to list repositories and tags, then reading image manifests and configuration to map what is stored before pulling images for secrets or planting a backdoor."
keywords:
  - registry enumeration
  - _catalog
  - tags list
  - registry v2 API
  - docker registry
---

# Registry enumeration

The registry v2 API exposes a catalog of repositories and the tags under each. Where the catalog is readable, it hands over the full inventory; even where it is not, known or guessed repository names can be queried for tags and manifests.

```bash
REG=https://registry.example.com
curl -s $REG/v2/_catalog | jq .                      # all repositories, if allowed
curl -s $REG/v2/<repo>/tags/list | jq .              # tags for a repository
curl -s $REG/v2/<repo>/manifests/<tag> \
  -H 'Accept: application/vnd.docker.distribution.manifest.v2+json' | jq '.config,.layers'
```

## Exploitation notes

- `_catalog` is often left readable on internal registries; it is the fastest full inventory.
- Manifests reference the config and layer digests you then pull for [Secrets in image layers](secrets-in-image-layers.md).
- Tools like regctl and crane wrap these calls and handle auth token exchange.

## References

- [Docker registry HTTP API v2](https://distribution.github.io/distribution/spec/api/)
- [OCI distribution specification](https://github.com/opencontainers/distribution-spec)
