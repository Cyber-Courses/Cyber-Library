---
title: "Secrets in image layers: recovering credentials baked into images"
description: "Recovering secrets committed into container image layers and build history: API keys, cloud credentials, SSH and TLS keys, and tokens that were copied in and later deleted but remain in an earlier layer, extracted from a pulled image."
keywords:
  - image layer secrets
  - docker history
  - deleted layer secret
  - dive trufflehog
  - credential recovery
---

# Secrets in image layers

Images are built up as layers, and a file deleted in a later layer still exists in the earlier one. Secrets copied in during a build, then removed, remain recoverable from the image. Build arguments and environment also persist in the image config and history.

```bash
docker pull <image> && docker history --no-trunc <image>   # commands, ARG/ENV values
docker save <image> -o img.tar && mkdir x && tar -xf img.tar -C x   # unpack layers

# Scan layers and history for secrets
dive <image>
trufflehog docker --image <image>
```

## Exploitation notes

- `docker history` exposes `ARG` and `ENV` values and the exact build commands, often enough on its own.
- Deleted-but-present files live in the layer tarballs under `x/`; search them for keys, `.npmrc`, `.git-credentials`, and cloud config.
- This is the payoff of pulling private images via [Registry access](registry-access.md) or an open registry.

## References

- [Docker: image layers and history](https://docs.docker.com/build/guide/layers/)
- [OCI image specification](https://github.com/opencontainers/image-spec)
