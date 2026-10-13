---
title: "BuildKit RCE : s'échapper du build pour lire l'hôte ou y exécuter du code"
description: "BuildKit résout le contexte de build, les frontends et les montages de cache à partir d'un Dockerfile influencé par l'attaquant. Des failles dans ce traitement ont permis à un build conçu de lire des fichiers hors du contexte de build via une traversée de chemin dans le cache ou les montages, de suivre des liens symboliques vers le système de fichiers de l'hôte, et dans certains cas d'exécuter du code sur l'hôte de build, compromettant un runner CI ou un serveur de build."
keywords:
  - buildkit
  - contexte de build
  - cache mount
  - path traversal
  - hôte de build
---

# BuildKit RCE

BuildKit est le backend de construction moderne de Docker. Il interprète un Dockerfile (ou un frontend alternatif), récupère le contexte de build et gère les directives de cache et de montage, le tout piloté par une entrée que l'attaquant contrôle lorsqu'il peut soumettre un Dockerfile ou un contexte de build, comme dans un système CI qui construit des dépôts utilisateur. Plusieurs classes de failles en découlent : un traitement des chemins de cache et de montage qui s'échappe du contexte prévu, un suivi de liens symboliques qui atteint des fichiers de l'hôte, et des problèmes d'analyse du frontend. L'impact va de la lecture de fichiers hors du contexte de build jusqu'à l'exécution de code sur l'hôte de build, qui en CI est souvent une cible riche en identifiants.

Identifiez le chemin de build et déterminez s'il est atteignable par l'attaquant :

```bash
docker buildx version 2>/dev/null; docker version | grep -i buildkit
# in CI: can an attacker submit the Dockerfile and context that gets built?
```

## Routes représentatives

```dockerfile
# 1. Cache or bind mount directives pointed outside the context, where a
#    traversal or symlink flaw lets the build read host files into a layer.
RUN --mount=type=cache,target=/c \
    cp /c/../../../../etc/shadow /out 2>/dev/null || true

# 2. A symlink in the build context that BuildKit follows during COPY,
#    resolving to a host path outside the context root.
#    (ship a symlink "ctx-link" -> / in the context, then COPY through it)
COPY ctx-link/etc/ /host-etc/
```

```bash
# 3. Frontend/daemon flaws: a crafted frontend reference or LLB that the
#    builder executes with builder privileges on the host. Version-specific;
#    fingerprint BuildKit and match to a known advisory.
```

Le mécanisme commun est qu'une directive censée être limitée au contexte de build est amenée à résoudre un chemin sur l'hôte de build, de sorte que du contenu traverse de l'hôte vers le build (une lecture) ou que le constructeur effectue une action dirigée par l'attaquant (une exécution).

## Notes d'exploitation

- La cible réaliste est un service de build partagé ou un runner CI qui construit des Dockerfiles non fiables ; là, l'hôte de build détient des identifiants de registre, des rôles cloud et les sources d'autres locataires.
- Ces failles sont spécifiques à une version ; prenez l'empreinte de BuildKit et faites-la correspondre à ses avis de sécurité plutôt que de supposer une primitive générique. Les versions corrigées contraignent la résolution des montages et des liens symboliques au contexte.
- Une primitive de lecture qui tire des fichiers de l'hôte dans une couche de sortie suffit souvent : volez les identifiants du runner, puis pivotez avec eux plutôt que d'avoir besoin d'une exécution complète.

## Références

- [BuildKit security advisories](https://github.com/moby/buildkit/security/advisories)
- [Docker: build context and mounts](https://docs.docker.com/build/building/context/)
