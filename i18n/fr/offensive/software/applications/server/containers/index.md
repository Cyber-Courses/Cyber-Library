---
title: "Conteneurs : techniques offensives contre les runtimes et l'orchestration"
order: 1
description: "Attaquer les charges de travail conteneurisées sur trois couches : l'évasion, indépendante du runtime, d'un conteneur vers son hôte, les runtimes de conteneurs et les plans de contrôle qu'ils exposent, et la couche d'orchestration qui planifie les conteneurs sur une flotte d'hôtes."
keywords:
  - sécurité des conteneurs
  - évasion de conteneur
  - Docker
  - Kubernetes
  - runtime de conteneur
---

# Conteneurs

Les conteneurs empaquettent une charge de travail et l'exécutent comme des processus hôtes ordinaires que le noyau isole à l'aide d'espaces de noms, de cgroups, de capacités et d'un profil seccomp ou LSM. Cette isolation est une configuration, et non une frontière imposée par le matériel, de sorte que le travail offensif contre les conteneurs porte surtout sur les endroits où l'isolation est faible, absente, ou rendue à la charge de travail à dessein.

Le sujet se divise en trois couches, chacune avec son propre modèle d'attaque.

## Sous-thèmes

- **[Évasion de conteneur](container-escape/index.md)** : sortir d'un conteneur unique vers son hôte. Ces primitives sont indépendantes du runtime : un drapeau privilégié, un montage dangereux, un espace de noms de l'hôte partagé, ou un exploit du runtime ou du noyau fonctionne de la même manière depuis un conteneur Docker, un conteneur Podman ou un pod Kubernetes.
- **[Runtimes](runtimes/index.md)** : attaquer les moteurs de conteneurs eux-mêmes plutôt que de s'évader d'un conteneur, à travers le démon ou les sockets d'API qu'ils exposent et la chaîne d'approvisionnement des images et registres derrière eux.
- **[Orchestration](orchestration/index.md)** : attaquer le plan de contrôle du cluster qui planifie les conteneurs sur les hôtes, de l'énumération et des composants exposés jusqu'au RBAC, à l'évasion de pod vers nœud, au mouvement latéral et à la persistance.

## Références

- [NCC Group : Understanding and Hardening Linux Containers](https://research.nccgroup.com/2016/04/13/understanding-and-hardening-linux-containers/)
- [Sécurité du moteur Docker](https://docs.docker.com/engine/security/)
- [Concepts de sécurité de Kubernetes](https://kubernetes.io/docs/concepts/security/)
