---
title: "Fuite de secret via build-arg : des identifiants qui survivent dans l'image construite"
description: "Les secrets passés à une construction avec --build-arg sont enregistrés dans l'historique de l'image et présents dans le système de fichiers de toute couche qui les a utilisés. Un attaquant capable de lire l'image résultante récupère les jetons, clés et mots de passe que l'auteur a transmis comme arguments de build en croyant qu'ils étaient éphémères."
keywords:
  - build-arg
  - docker history
  - build secret
  - identifiants intégrés
  - couches d'image
---

# Fuite de secret via build-arg

Les développeurs passent souvent un secret à une construction avec `--build-arg TOKEN=...` pour cloner un dépôt privé ou télécharger une dépendance, en supposant qu'il disparaît à la fin du build. Ce n'est pas le cas. La valeur `ARG` est enregistrée dans l'historique de construction de l'image, et tout fichier créé avec elle se trouve dans cette couche. Quiconque peut lire l'image, via un pull depuis un registre ou un conteneur en cours d'exécution, récupère le secret.

## Récupérer la valeur

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

Le champ `created_by` de l'historique de configuration est littéralement la ligne de commande qui a été exécutée, y compris l'argument de build interpolé, de sorte qu'un `RUN git clone https://$TOKEN@...` expose le jeton textuellement.

## Notes d'exploitation

- `--build-arg` est la fuite ; le mécanisme correct de secret de build (`RUN --mount=type=secret`) ne persiste pas, mais beaucoup d'images sont antérieures à ce mécanisme ou l'ignorent, donc l'exploration de l'historique reste productive.
- Vérifiez à la fois l'historique (`created_by`) et le système de fichiers de la couche extraite : un jeton peut apparaître dans la ligne de commande, dans un fichier `.netrc` ou `.git-credentials`, ou dans un téléchargement mis en cache.
- Les identifiants de build récupérés ont souvent une portée de dépôt ou de registre qui débloque le code source ou d'autres images ; voir [Secrets dans les couches d'image](../images-and-registries/secrets-in-image-layers.md) pour le flux de travail général d'exploration des couches.

## Références

- [Docker: build secrets vs build args](https://docs.docker.com/build/building/secrets/)
- [Docker: image history](https://docs.docker.com/reference/cli/docker/image/history/)
