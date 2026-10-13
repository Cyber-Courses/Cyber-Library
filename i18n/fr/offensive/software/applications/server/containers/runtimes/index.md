---
order: 2
title: "Runtimes : attaquer les moteurs qui construisent et exécutent les conteneurs"
description: "Les runtimes de conteneurs eux-mêmes constituent une surface d'attaque distincte de la frontière du conteneur : le moteur Docker et son API réseau, les runtimes de nœud Kubernetes containerd et CRI-O avec leurs sockets de contrôle, et le modèle rootless et à activation par socket de Podman. Chacun expose des plans de contrôle, des identifiants et une gestion d'images qui mènent à la compromission de l'hôte ou du nœud."
keywords:
  - runtime de conteneur
  - docker
  - containerd
  - cri-o
  - podman
---

# Runtimes

Un runtime de conteneur fait bien plus que démarrer des processus : il expose une API de contrôle, récupère et stocke des images avec des identifiants de registre en cache, construit des images et s'exécute en root (ou quasi-root) sur l'hôte. Cette machinerie est une surface d'attaque à part entière, distincte de l'évasion de la frontière du conteneur, qui est une propriété liée au partage du noyau documentée dans l'évasion de conteneur. Les runtimes diffèrent par la manière dont ils exposent cette surface : Docker via une API de démon équivalente à root, containerd et CRI-O via des sockets de contrôle de nœud sous Kubernetes, et Podman via une conception délibérément rootless et à activation par socket.

```bash
# identify the runtimes present and their control surfaces
docker info 2>/dev/null | grep -iE 'rootless|server version'
ls -l /run/containerd/containerd.sock /run/crio/crio.sock 2>/dev/null
ls -l /run/podman/podman.sock /run/user/*/podman/podman.sock 2>/dev/null
ps -ef | grep -E 'dockerd|containerd|crio|podman' | grep -v grep
```

## Sous-thèmes

- **[Docker](docker/index.md)** : l'API du moteur, les images et registres, et le processus de build.
- **[containerd et CRI-O](containerd-and-cri-o/index.md)** : les runtimes de nœud Kubernetes et leurs sockets.
- **[Podman](podman/index.md)** : le modèle rootless, sans démon, à activation par socket.

## Références

- [Docker: security](https://docs.docker.com/engine/security/)
- [Kubernetes: container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
- [Podman documentation](https://docs.podman.io/)
