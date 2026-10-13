---
order: 1
title: "Secrets dans les couches d'image : extraire les identifiants figés lors d'un build"
description: "Chaque instruction d'un Dockerfile crée une couche immuable, donc un secret ajouté à une étape puis supprimé à une étape ultérieure existe toujours dans la couche antérieure. Un attaquant décompresse les couches d'une image et son historique de build pour récupérer des clés d'API, des clés privées et des mots de passe que l'auteur croyait supprimés, avec pour seul prérequis un accès en lecture à l'image."
keywords:
  - couches d'image
  - docker history
  - secrets figés
  - dockerfile
  - vol d'identifiants
---

# Secrets dans les couches d'image

Une image de conteneur est une pile ordonnée de couches immuables, une par instruction de build. Supprimer un fichier dans une couche ultérieure ne fait qu'ajouter une entrée de type whiteout ; le fichier existe toujours dans la couche qui l'a créé. Tout secret copié, écrit dans un fichier par echo, ou passé comme argument de build puis supprimé reste donc récupérable depuis l'image. Un accès en lecture à l'image, via un pull ou un registre, est tout ce qui est nécessaire pour l'extraire.

## Lire l'historique et les couches

```bash
# build history shows ENV, ARG, and RUN steps, including echoed secrets
docker history --no-trunc <image>
# save the image and unpack every layer to search the real filesystem contents
docker save <image> -o img.tar && mkdir img && tar -xf img.tar -C img
for l in img/*/layer.tar img/blobs/sha256/*; do tar -xf "$l" -C extracted 2>/dev/null; done
grep -rIER 'AKIA|BEGIN (RSA|OPENSSH) PRIVATE KEY|password|secret|token' extracted | head
```

Décompresser les couches (plutôt que d'inspecter uniquement la vue fusionnée du conteneur en cours d'exécution) est ce qui permet de récupérer des fichiers qu'une couche ultérieure a supprimés, car chaque archive tar de couche les contient encore.

## Sources fréquentes de secrets figés

```bash
# build args passed with --build-arg are visible in history
docker history --no-trunc <image> | grep -i 'ARG\|ENV'
# credential files COPYed in then removed survive in the adding layer
# .git directories, .env files, and SSH keys are frequent finds
find extracted -name '.env' -o -name 'id_*' -o -name '.netrc' -o -path '*/.git/config'
```

## Notes d'exploitation

- Décompressez et parcourez toujours les couches, pas seulement le système de fichiers final : la vue fusionnée masque les secrets supprimés que les archives tar par couche conservent.
- Les arguments de build sont une fuite fréquente, car les auteurs les considèrent comme transitoires, alors qu'ils sont enregistrés en clair dans l'historique de l'image ; les secrets passés via `--build-arg` sont récupérables par quiconque peut lire l'image.
- Les clés cloud et clés privées récupérées débloquent souvent bien plus que le conteneur ; triez-les en premier. À combiner avec [Accès au registre](registry-access.md) pour atteindre des images privées à grande échelle.

## Outils

- [dive (explorateur de couches)](https://github.com/wagoodman/dive)
- [trufflehog (recherche de secrets, prend en charge les images docker)](https://github.com/trufflesecurity/trufflehog)

## Références

- [Docker: image layers and history](https://docs.docker.com/build/guide/layers/)
- [OWASP: secrets in container images](https://owasp.org/www-project-docker-top-10/)
