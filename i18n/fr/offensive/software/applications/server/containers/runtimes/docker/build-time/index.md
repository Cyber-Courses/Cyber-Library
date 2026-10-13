---
order: 3
title: "Attaques au moment du build : abuser du processus de construction d'image"
description: "La construction d'un conteneur est un environnement d'exécution doté de sa propre surface d'attaque. Les arguments de build et les secrets montés fuient dans l'image finale ou dans le cache de build, et le frontend BuildKit ainsi que le daemon ont présenté des failles permettant à un Dockerfile ou un contexte de build conçu de lire des fichiers hors du contexte ou d'exécuter du code sur l'hôte de build."
keywords:
  - docker build
  - buildkit
  - build-arg
  - build secret
  - hôte de build
---

# Attaques au moment du build

La construction d'une image exécute des commandes, résout un contexte et met en cache des résultats intermédiaires, le tout sur un hôte de build qui dispose fréquemment de plus d'accès que l'environnement d'exécution final : identifiants de registre, rôles cloud pour les push et le dépôt de code source. Deux problèmes en découlent. Les secrets fournis au build fuient dans l'image ou le cache où ils peuvent être récupérés plus tard. Et l'outillage de build lui-même, BuildKit et le daemon, traite un Dockerfile et un contexte influencés par l'attaquant, de sorte que des failles permettent de lire des fichiers de l'hôte ou d'exécuter du code sur l'hôte de build.

```bash
docker version | grep -i buildkit            # BuildKit frontend in use
env | grep -iE 'BUILD|REGISTRY|AWS|GITHUB_TOKEN'   # build-host credentials worth stealing
```

## Sous-thèmes

- **[Fuite de secret via build-arg](build-arg-secret-leak.md)** : des secrets passés comme arguments de build persistant dans l'image.
- **[BuildKit RCE](buildkit-rce.md)** : des builds conçus lisant hors du contexte ou exécutant du code sur l'hôte de build.

## Références

- [Docker: build secrets](https://docs.docker.com/build/building/secrets/)
- [BuildKit security advisories](https://github.com/moby/buildkit/security/advisories)
