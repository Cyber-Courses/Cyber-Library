---
title: "Configuration privilégiée : s'évader via des réglages de conteneur trop permissifs"
order: 2
description: "Évasion de conteneur via une configuration qui confie trop de pouvoir à la charge de travail : le flag privileged tout-en-un, des capacités Linux dangereuses individuelles, l'accès direct aux périphériques de l'hôte, un profil seccomp ou AppArmor désactivé, et un release_agent de cgroup accessible en écriture. Chacune est une voie concrète, sans exploit, de l'intérieur d'un conteneur vers l'exécution de code sur l'hôte."
keywords:
  - conteneur privilégié
  - capacités linux
  - seccomp apparmor
  - cgroup release_agent
  - évasion de conteneur
---

# Configuration privilégiée

Les évasions de conteneur les plus courantes ne nécessitent aucun exploit de corruption mémoire : le conteneur a simplement été démarré avec plus de pouvoir que l'isolation ne peut en survivre, et l'attaquant utilise ce pouvoir tel qu'il a été conçu. Avant de tenter quoi que ce soit, identifiez exactement ce qui vous a été accordé, car cela détermine quelle voie fonctionne.

```bash
# Effective capability set (the single most important check)
grep CapEff /proc/self/status
# => CapEff: 000001ffffffffff  is the FULL set = --privileged; decode any value with:
capsh --decode=000001ffffffffff

# Seccomp mode: 0 = disabled, 2 = a filter is loaded
grep Seccomp /proc/self/status
# AppArmor/SELinux confinement for this process
cat /proc/self/attr/current 2>/dev/null        # "unconfined" means no AppArmor profile

# Device access and the cgroup version/writability
ls -l /dev | head                                # a long device list hints at --privileged
cat /proc/1/cgroup; mount | grep cgroup          # v1 (named hierarchies) vs v2 (unified)

# User namespace: is container root real host root?
cat /proc/self/uid_map                           # "0 0 4294967295" => UID 0 == host UID 0
```

Lisez cette sortie de haut en bas : un `CapEff` complet, une liste de périphériques visible et `Seccomp: 0` signifient ensemble `--privileged`, et toutes les voies ci-dessous sont ouvertes. Une seule capacité ajoutée vous réduit à sa primitive spécifique. Un uid_map `0 0 ...` signifie que tout ce que vous obtenez s'exécute en tant que vrai root de l'hôte ; une plage mappée le confine.

## Sous-sujets

- **[Flag privileged](privileged-flag.md)** : `--privileged`, qui accorde tout ce qui suit d'un coup.
- **[Abus de capacité](capability-abuse/index.md)** : une seule capacité dangereuse, chacune avec sa propre voie vers l'hôte.
- **[Accès aux périphériques](device-access/index.md)** : des périphériques bruts de l'hôte exposés dans le conteneur.
- **[Seccomp ou AppArmor non confinés](unconfined-seccomp-or-apparmor.md)** : un profil d'appels système et de LSM affaibli.
- **[cgroups release_agent](cgroups-release-agent.md)** : un release_agent accessible en écriture qui exécute un programme sur l'hôte.

## Références

- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Docker : runtime privilege and Linux capabilities](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)
- [Trail of Bits : Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
