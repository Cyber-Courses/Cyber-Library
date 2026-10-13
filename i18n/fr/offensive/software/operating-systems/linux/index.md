---
title: "Linux : attaquer la plateforme hôte Linux"
description: "Techniques offensives contre un hôte Linux : élévation de privilèges locale via les binaires SUID et SGID, les règles sudo, les capabilities Linux, cron, les chemins inscriptibles et l'abus du PATH, accès aux identifiants et secrets, abus des frontières de conteneurs et de namespaces, et exploitation du noyau."
keywords:
  - élévation de privilèges Linux
  - SUID
  - sudo
  - capabilities Linux
  - exploitation du noyau
---

# Linux

Linux est la plateforme de serveurs et d'infrastructure dominante, et sa surface d'attaque locale découle du modèle de privilèges Unix : un unique super-utilisateur `root`, des règles de permissions et de propriété de fichiers, et quelques mécanismes qui accordent délibérément une exécution élevée. Une fois que du code s'exécute en tant qu'utilisateur non privilégié, le travail consiste à trouver l'un de ces mécanismes mal configuré et à s'en servir pour atteindre `root`, puis à récolter les secrets que l'hôte détient.

## La surface locale

- **Élévation de privilèges** : binaires SUID/SGID (et les GTFOBins qui en abusent), règles `sudo` permissives, **capabilities** de fichiers trop larges, unités `cron` et systemd inscriptibles, détournement du `PATH` et des bibliothèques (`LD_PRELOAD`/`LD_LIBRARY_PATH`), et fichiers sensibles inscriptibles (`/etc/passwd`, `/etc/shadow`, sudoers).
- **Accès aux identifiants et secrets** : fichiers d'historique, configurations et clés lisibles par tous, clés et agents SSH, identifiants de services, et secrets en mémoire.
- **Frontières de conteneurs et de namespaces** : s'échapper d'un conteneur vers son hôte est un domaine approfondi à part entière dans [Containers](../../applications/server/containers/index.md) ; les primitives de namespace, de cgroup et de capability dont il abuse sont les mêmes que celles qui gouvernent les privilèges ici.
- **Noyau** : exploitation locale du noyau (les primitives classiques d'écrasement et d'élévation) pour l'étape finale vers `root`.

## Jonctions

L'évasion de conteneur est couverte dans [Containers](../../applications/server/containers/index.md), et l'évasion d'invité à hôte d'un hyperviseur dans [Virtualization](../../applications/server/virtualization/index.md) ; les deux sont référencées. Ce domaine porte sur l'hôte Linux lui-même.

## Références

- [GTFOBins](https://gtfobins.github.io/)
- [MITRE ATT&CK: privilege escalation](https://attack.mitre.org/tactics/TA0004/)
