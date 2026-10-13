---
order: 2
title: "Podman : attaquer un runtime rootless et sans démon"
description: "Podman exécute des conteneurs sans démon central et fonctionne par défaut en mode rootless, ce qui modifie sa surface d'attaque. Le socket API optionnel accorde tout de même le contrôle des conteneurs, le modèle rootless fondé sur les user namespaces détermine ce qu'une évasion rapporte réellement, et les outils associés skopeo et buildah gèrent les images et les builds avec leur propre exposition d'identifiants."
keywords:
  - podman
  - conteneurs rootless
  - socket podman
  - skopeo
  - buildah
---

# Podman

Podman est un moteur de conteneurs conçu pour fonctionner sans démon central privilégié et, par défaut, sans root. Cela remodèle sa surface offensive. Il n'y a pas de démon root permanent à prendre pour cible, mais Podman peut exposer un socket API compatible Docker qui accorde le contrôle des conteneurs ; son modèle rootless, construit sur les user namespaces, détermine si une évasion de conteneur aboutit à un utilisateur non privilégié ou à root ; et ses outils associés, `skopeo` pour le transport d'images et `buildah` pour les builds, présentent la même exposition d'identifiants de registre et de secrets d'image que Docker.

```bash
podman info 2>/dev/null | grep -iE 'rootless|graphRoot|runRoot'
ls -l /run/podman/podman.sock /run/user/*/podman/podman.sock 2>/dev/null
cat /proc/self/uid_map                               # rootless => mapped range, not 0 0
```

## Sous-thèmes

- **[Modèle rootless](rootless-model.md)** : comment la conception à user namespace contraint ou permet les évasions.
- **[Socket de service API](api-service-socket.md)** : le socket de contrôle optionnel compatible Docker.
- **[Activation par socket systemd](systemd-socket-activation.md)** : l'activation à la demande de l'API et son exposition.
- **[skopeo et buildah](skopeo-and-buildah.md)** : exposition d'identifiants des outils de transport d'images et de build.

## Références

- [Podman documentation](https://docs.podman.io/)
- [Podman: rootless containers](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)
- [Rootless containers](https://rootlesscontaine.rs/)
