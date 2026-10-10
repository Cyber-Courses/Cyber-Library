---
title: "Montages sensibles : s'évader via des chemins hôtes et des interfaces noyau montés en bind dans un conteneur"
order: 1
description: "Évasions de conteneur provenant d'un chemin hôte exposé à l'intérieur du conteneur : un montage bind d'un répertoire hôte ou de la racine complète de l'hôte, un socket de contrôle de runtime monté tel que docker.sock, et des interfaces noyau procfs ou sysfs accessibles en écriture comme core_pattern, modprobe et uevent_helper que le noyau de l'hôte exécute en tant que root."
keywords:
  - montages sensibles
  - docker socket
  - core_pattern
  - hostpath
  - évasion de conteneur
---

# Montages sensibles

De nombreuses évasions ne nécessitent aucune astuce de capacité : le conteneur s'est simplement vu attribuer un chemin hôte qu'il ne devrait pas avoir. Ce chemin peut être un montage bind de répertoire explicite, la racine complète de l'hôte, un socket de contrôle de runtime, ou un pseudo-fichier noyau sous `/proc` et `/sys` que le noyau de l'hôte relit et sur lequel il agit en tant que root. Le point commun est que l'objet de l'autre côté du montage appartient à l'hôte, de sorte que l'écrire atteint l'hôte directement.

Énumérez les montages et recherchez les sources appartenant à l'hôte :

```bash
cat /proc/self/mountinfo                 # full mount table with source subtrees
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS  # readable summary; host bind sources stand out
# control sockets and sensitive kernel files
ls -l /var/run/docker.sock /run/containerd/containerd.sock /run/crio/crio.sock 2>/dev/null
ls -l /proc/sys/kernel/core_pattern /proc/sys/kernel/modprobe /sys/kernel/uevent_helper 2>/dev/null
mount | grep -E '/proc|/sys' | grep -v 'ro,'   # writable proc/sys indicates host interfaces
```

Un montage bind hôte affiche un sous-arbre hôte comme source ; un `/proc/sys/kernel/*` accessible en écriture ou un socket de runtime constitue une voie directe.

## Sous-thèmes

- **[Montage de chemin hôte](host-path-mount.md)** : un montage bind d'un répertoire hôte ou du système de fichiers racine.
- **[Montage de socket de runtime](runtime-socket-mount.md)** : un socket de contrôle Docker, containerd ou CRI-O monté.
- **[procfs et sysfs](procfs-and-sysfs/index.md)** : interfaces noyau accessibles en écriture que l'hôte exécute en tant que root.

## Références

- [BishopFox : bad pod and sensitive mounts](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [HackTricks : sensitive mounts](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
