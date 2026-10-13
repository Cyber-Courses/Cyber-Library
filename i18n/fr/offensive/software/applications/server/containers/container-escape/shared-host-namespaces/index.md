---
title: "Espaces de noms de l'hôte partagés : évasions depuis un conteneur qui partage les espaces de noms de l'hôte"
order: 4
description: "Chaque drapeau d'espace de noms --host supprime une couche d'isolation entre le conteneur et l'hôte. Partager l'espace de noms PID expose les processus de l'hôte et permet d'y injecter, l'espace de noms réseau expose les services locaux à l'hôte et le loopback du nœud, l'espace de noms IPC expose la mémoire partagée et les sémaphores, et un espace de noms utilisateur partagé ou mappé peut signifier que le root du conteneur est le root de l'hôte."
keywords:
  - espace de noms de l'hôte
  - hostpid hostnetwork
  - espace de noms pid
  - espace de noms utilisateur
  - évasion de conteneur
---

# Espaces de noms de l'hôte partagés

Les espaces de noms sont ce qui fait d'un conteneur un conteneur : des vues séparées des processus, du réseau, de l'IPC, des montages, des utilisateurs, et plus encore. Chaque drapeau `--pid=host`, `--net=host`, `--ipc=host` ou `--userns=host` (et les champs Kubernetes `hostPID`, `hostNetwork`, `hostIPC`) supprime l'une de ces séparations, rattachant le conteneur à la vue de l'hôte. Aucun d'eux n'est un exploit de corruption mémoire ; chacun rend simplement une tranche de l'hôte, et l'évasion consiste à utiliser cette tranche.

Identifiez quels espaces de noms sont partagés avec l'hôte en comparant les identifiants d'espace de noms avec ceux du PID 1 sur l'hôte, ou par ce qui est visible :

```bash
ls -l /proc/self/ns/                       # this process's namespace inode IDs
ls -l /proc/1/ns/                          # compare: same inode = shared with host init
ls /proc | grep -E '^[0-9]+$' | wc -l      # many host PIDs => shared PID namespace
ip addr; ss -tlnp 2>/dev/null | head       # host interfaces/ports => shared net namespace
ipcs                                       # host shared memory/semaphores => shared IPC
cat /proc/self/uid_map                     # "0 0 ..." => not in a user namespace
```

## Sous-thèmes

- **[Espace de noms PID de l'hôte](host-pid-namespace.md)** : voir les processus de l'hôte et y injecter.
- **[Espace de noms réseau de l'hôte](host-network-namespace.md)** : atteindre les services locaux à l'hôte et intercepter le trafic.
- **[Espace de noms IPC de l'hôte](host-ipc-namespace.md)** : lire la mémoire partagée et les files de messages.
- **[Espace de noms utilisateur de l'hôte](host-user-namespace.md)** : quand le root du conteneur est mappé vers le root de l'hôte.

## Références

- [man 7 namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)
- [Kubernetes : sécurité des pods et espaces de noms de l'hôte](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [HackTricks : namespaces](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces)
