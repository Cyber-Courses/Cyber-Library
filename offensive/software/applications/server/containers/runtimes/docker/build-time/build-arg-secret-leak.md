---
title: "Build-arg secret leak: recovering secrets from build arguments and layers"
description: "Recovering secrets that were passed to a Docker build as build arguments or written into intermediate layers, which persist in the image config and history even when a later step deletes the file, exposing credentials to anyone who pulls the image."
keywords:
  - build-arg
  - ARG secret
  - docker history
  - intermediate layer
  - credential leak
---

# Build-arg secret leak

Passing a secret with `--build-arg` or `ARG` bakes it into the image: `ARG` values are recorded in the image history, and anything written to the filesystem during a `RUN` persists in that layer even if a later step removes it. The result is credentials readable by anyone who pulls the image.

```bash
docker history --no-trunc <image> | grep -iE 'ARG|token|key|secret'
# Files written then deleted still live in the layer tarballs
docker save <image> -o img.tar && tar -xf img.tar && grep -rniE 'BEGIN PRIVATE KEY|aws_secret' .
```

## Exploitation notes

- `ARG` is not a secret mechanism; its value is visible in `docker history`. The intended mechanism is BuildKit `--mount=type=secret`, which does not persist.
- The highest-value leaks are cloud keys, registry credentials, and private deploy keys baked during dependency installation.
- This overlaps [Secrets in image layers](../images-and-registries/secrets-in-image-layers.md); the difference is the build-time origin.

## References

- [Docker build secrets](https://docs.docker.com/build/building/secrets/)
- [Docker: image history](https://docs.docker.com/reference/cli/docker/image/history/)
