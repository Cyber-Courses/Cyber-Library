---
title: "Registry enumeration: listing repositories, tags, and manifests"
description: "The OCI distribution API exposes catalog, tag, and manifest endpoints that reveal what a registry holds. An attacker lists every repository and tag, pulls manifests to map layers, and identifies images likely to contain secrets or to be widely consumed, all before authenticating if the registry allows anonymous reads."
keywords:
  - docker registry
  - registry api v2
  - catalog
  - manifest
  - enumeration
---

# Registry enumeration

A registry speaks the OCI distribution API (the Docker Registry HTTP API v2), which has well-known endpoints for listing content. Many registries allow anonymous reads, so enumeration frequently succeeds with no credentials and maps the entire holdings: which repositories exist, what tags each has, and which layers make up an image. This is the reconnaissance that drives pulling the right image for secrets or picking a widely used one to poison.

```bash
R=http://<registry>:5000
# list all repositories
curl -s $R/v2/_catalog | jq .
# list tags for a repository
curl -s $R/v2/<repo>/tags/list | jq .
# fetch the manifest (needs the v2 manifest Accept header) to see layers and config
curl -s -H 'Accept: application/vnd.docker.distribution.manifest.v2+json' \
  $R/v2/<repo>/manifests/<tag> | jq .
# pull the config blob, which holds env, entrypoint, and build history
curl -s $R/v2/<repo>/blobs/<config-digest> | jq '.config.Env, .history'
```

The manifest lists layer digests and the config digest; the config blob exposes environment variables, the entrypoint, and the layer-by-layer history, which is where secrets and sensitive build steps appear.

## Exploitation notes

- Anonymous catalog and manifest reads are the default for the open-source registry unless an auth proxy is placed in front; a `200` on `/v2/_catalog` means the holdings are enumerable.
- Prioritise repositories whose names suggest internal or build images (`ci`, `internal`, `base`, app names), which are the most likely to contain credentials or to be widely consumed.
- The config blob's `Env` and `history` often reveal secrets without downloading any layer; pull layers only for the images that look promising. Continue with [Secrets in image layers](secrets-in-image-layers.md).

## Tools

- [regctl (registry client)](https://github.com/regclient/regclient)
- [DockerRegistryGrabber](https://github.com/Syzik/DockerRegistryGrabber)

## References

- [Docker Registry HTTP API V2](https://distribution.github.io/distribution/spec/api/)
- [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec/blob/main/spec.md)
