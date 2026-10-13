---
order: 1
title: "API du daemon exposée : contrôler Docker sur le réseau"
description: "Le daemon Docker expose une API de contrôle complète. Lorsqu'elle est liée à un port TCP, atteignable sans authentification sur 2375 ou protégée seulement par un TLS faible sur 2376, quiconque peut l'atteindre dispose d'un contrôle équivalent à root sur l'hôte : il énumère l'environnement puis lance un conteneur privilégié qui monte le système de fichiers de l'hôte."
keywords:
  - daemon docker
  - docker api
  - port 2375
  - docker distant
  - prise de contrôle de l'hôte
---

# API du daemon exposée

L'API du daemon Docker est toute sa surface de contrôle, et elle est équivalente à root par conception : quiconque peut l'appeler peut démarrer un conteneur qui monte l'hôte et s'exécute en tant que root. Localement, l'API vit sur une socket Unix, mais les opérateurs l'exposent fréquemment sur TCP pour la gestion à distance, sur 2375 (HTTP en clair, sans authentification) ou 2376 (TLS). Un 2375 exposé, ou un 2376 avec un TLS faible ou optionnel, remet le contrôle complet de l'hôte à tout client capable d'atteindre le port, sans exploit requis.

Trouver et sonder un daemon exposé :

```bash
# scan for the Docker API ports
nmap -p 2375,2376 --open <target-range>
# a single unauthenticated request confirms control
curl -s http://<target>:2375/version
curl -s http://<target>:2375/info | head
```

Un `200` avec la version et les infos du daemon depuis 2375 signifie un contrôle sans authentification.

## Sous-thèmes

- **[Accès non authentifié au daemon](unauthenticated-daemon-access.md)** : un 2375 ouvert sans authentification.
- **[TLS faible sur 2376](weak-tls-on-2376.md)** : TLS présent mais mal configuré ou optionnel.
- **[Énumération de l'API](api-enumeration.md)** : cartographier les conteneurs, images, secrets et réseaux via l'API.
- **[Prise de contrôle de l'hôte via un run privilégié](host-takeover-via-privileged-run.md)** : lancer un conteneur qui s'empare de l'hôte.

## Références

- [Docker Engine API reference](https://docs.docker.com/reference/api/engine/)
- [Docker: protect the daemon socket](https://docs.docker.com/engine/security/protect-access/)
- [HackTricks: 2375 Docker](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
