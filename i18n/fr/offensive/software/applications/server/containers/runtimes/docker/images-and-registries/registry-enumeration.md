---
order: 4
title: "Énumération de registre : lister les dépôts, tags et manifestes"
description: "L'API de distribution OCI expose des endpoints catalog, tag et manifest qui révèlent le contenu d'un registre. Un attaquant liste chaque dépôt et chaque tag, récupère les manifestes pour cartographier les couches et identifie les images susceptibles de contenir des secrets ou d'être largement consommées, le tout avant toute authentification si le registre autorise les lectures anonymes."
keywords:
  - docker registry
  - registry api v2
  - catalogue
  - manifeste
  - énumération
---

# Énumération de registre

Un registre parle l'API de distribution OCI (l'API HTTP v2 du Docker Registry), qui dispose d'endpoints bien connus pour lister le contenu. De nombreux registres autorisent les lectures anonymes, si bien que l'énumération réussit fréquemment sans aucun identifiant et cartographie l'ensemble des avoirs : quels dépôts existent, quels tags chacun possède et quelles couches composent une image. C'est la reconnaissance qui permet ensuite de récupérer la bonne image pour y chercher des secrets ou d'en choisir une largement utilisée pour l'empoisonner.

```bash
R=http://<registry>:5000
# list all repositories
curl -s $R/v2/_catalog | jq .
# list tags for a repository
curl -s $R/v2/<repo>/tags/list | jq .
# fetch the manifest (needs the v2 manifest Accept header) to see layers and config
curl -s -H 'Accept: application/vnd.docker.distribution.manifest.v2+json' \
  $R/v2/<repo>/manifests/<tag> | jq .
# pull the config blob, which holds env, entrypoint, and build history
curl -s $R/v2/<repo>/blobs/<config-digest> | jq '.config.Env, .history'
```

Le manifeste liste les digests des couches ainsi que le digest de la configuration ; le blob de configuration expose les variables d'environnement, l'entrypoint et l'historique couche par couche, qui est l'endroit où apparaissent les secrets et les étapes de build sensibles.

## Notes d'exploitation

- Les lectures anonymes du catalogue et des manifestes sont le comportement par défaut du registre open source à moins qu'un proxy d'authentification ne soit placé en amont ; un `200` sur `/v2/_catalog` signifie que les avoirs sont énumérables.
- Priorisez les dépôts dont les noms suggèrent des images internes ou de build (`ci`, `internal`, `base`, noms d'applications), qui sont les plus susceptibles de contenir des identifiants ou d'être largement consommés.
- Les champs `Env` et `history` du blob de configuration révèlent souvent des secrets sans télécharger aucune couche ; ne récupérez les couches que pour les images qui semblent prometteuses. Poursuivez avec [Secrets dans les couches d'image](secrets-in-image-layers.md).

## Outils

- [regctl (client de registre)](https://github.com/regclient/regclient)
- [DockerRegistryGrabber](https://github.com/Syzik/DockerRegistryGrabber)

## Références

- [Docker Registry HTTP API V2](https://distribution.github.io/distribution/spec/api/)
- [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec/blob/main/spec.md)
