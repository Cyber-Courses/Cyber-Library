---
order: 1
title: "Docker : attaquer le moteur, son API, ses images et ses builds"
description: "Surface offensive du moteur Docker : une API de démon équivalente à root et souvent exposée sur le réseau, des images et des registres qui divulguent des secrets et comportent un risque sur la chaîne d'approvisionnement, et un processus de build qui fuit des identifiants et présente ses propres vulnérabilités. L'évasion de conteneur elle-même est indépendante du runtime et documentée séparément."
keywords:
  - docker
  - démon docker
  - docker registry
  - docker build
  - sécurité des conteneurs
---

# Docker

Docker est le moteur de conteneurs le plus largement déployé, et sa surface offensive est le moteur et ses services de support plutôt que la frontière du conteneur, qui est une propriété du noyau partagée avec tous les autres runtimes. L'API du démon est équivalente à root et fréquemment joignable sur le réseau ; les images et les registres détiennent des secrets et constituent une cible de chaîne d'approvisionnement ; et le processus de build est un environnement d'exécution qui fuit des identifiants. S'évader d'un conteneur Docker vers l'hôte utilise les mêmes primitives que n'importe quel runtime et est couvert par l'évasion de conteneur.

```bash
docker version 2>/dev/null; docker info 2>/dev/null | grep -iE 'rootless|security'
ss -tlnp 2>/dev/null | grep -E '2375|2376'          # daemon exposed over TCP
ls -l /var/run/docker.sock 2>/dev/null               # local socket reachable
```

## Sous-thèmes

- **[API de démon exposée](exposed-daemon-api/index.md)** : contrôler le démon équivalent à root sur le réseau.
- **[Images et registres](images-and-registries/index.md)** : secrets dans les images et attaques de chaîne d'approvisionnement sur les registres.
- **[Attaques au moment du build](build-time/index.md)** : fuite de secrets et exploitation du processus de build.

## Références

- [Docker Engine API reference](https://docs.docker.com/reference/api/engine/)
- [Docker: security](https://docs.docker.com/engine/security/)
- [HackTricks: Docker security](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security)
