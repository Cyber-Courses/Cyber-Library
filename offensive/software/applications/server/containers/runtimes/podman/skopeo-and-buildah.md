---
title: "skopeo and buildah: abusing Podman's companion image tools"
description: "Abusing skopeo and buildah, Podman's companion tools for copying, inspecting, and building images, to reach registries with the user's stored credentials, pull and inspect private images, and build or push backdoored ones."
keywords:
  - skopeo
  - buildah
  - registry credentials
  - image inspection
  - podman tools
---

# skopeo and buildah

Podman environments ship `skopeo` (copy and inspect images between registries and local storage) and `buildah` (build images). Both read the same credential stores Podman uses, so on a host where they are configured they are a ready path to the registries that user can reach, with no daemon involved.

```bash
# Inspect and copy images using the user's stored auth
skopeo inspect docker://registry.example.com/<repo>:<tag>
skopeo copy docker://registry.example.com/<repo>:<tag> dir:/tmp/img

# Build and push a modified image with buildah
buildah from <repo>:<tag>; buildah run <ctr> -- sh -c 'echo backdoor'; buildah commit <ctr> <repo>:<tag>
```

## Exploitation notes

- Credentials come from `${XDG_RUNTIME_DIR}/containers/auth.json` or `~/.docker/config.json`; recovering that file is registry access.
- `skopeo copy` moves images without a daemon or a full pull, which is quiet and works from constrained hosts.
- Push rights through these tools enable [Image backdooring](../docker/images-and-registries/image-backdooring.md).

## References

- [skopeo](https://github.com/containers/skopeo)
- [buildah](https://buildah.io/)
