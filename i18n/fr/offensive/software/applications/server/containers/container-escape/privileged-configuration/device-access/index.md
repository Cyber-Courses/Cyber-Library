---
title: "Accès aux périphériques : s'évader via des nœuds de périphérique de l'hôte exposés dans un conteneur"
description: "Lorsqu'un conteneur peut voir des nœuds de périphérique de l'hôte sous /dev, que ce soit via --privileged ou un --device explicite, les pilotes de périphérique du noyau derrière ces nœuds opèrent sur le matériel de l'hôte quels que soient les espaces de noms. Monter le périphérique bloc de l'hôte, lire les périphériques de mémoire brute du noyau, ou créer de nouveaux nœuds de périphérique avec mknod mènent chacun du conteneur vers l'hôte."
keywords:
  - accès aux périphériques de conteneur
  - répertoire dev
  - périphérique bloc de l'hôte
  - mknod
  - évasion de conteneur
---

# Accès aux périphériques

Les nœuds de périphérique sont la ligne directe du conteneur vers le matériel de l'hôte. Un nœud de périphérique n'est qu'une référence à un pilote identifié par un numéro majeur et un numéro mineur ; le pilote lui-même s'exécute dans le noyau hôte partagé et agit sur le périphérique réel, de sorte qu'un nœud de périphérique bloc dans un conteneur lit le véritable disque de l'hôte, et non une copie cloisonnée par espace de noms. Un conteneur obtient des nœuds de périphérique dangereux via `--privileged` (qui ajoute une règle de cgroup de périphériques autorisant tous les périphériques) ou via un `--device=/dev/sda` ciblé, et dans Kubernetes via une revendication de bloc brut `volumeDevices`.

Énumérez ce qui est accessible :

```bash
ls -l /dev                              # long list with sdX/nvme/mem hints at broad access
cat /sys/fs/cgroup/devices/devices.list 2>/dev/null   # cgroup v1 device allowlist; "a *:* rwm" = all
lsblk 2>/dev/null; cat /proc/partitions # host block devices and sizes
```

Une ligne `a *:* rwm` dans le cgroup de périphériques, ou des nœuds `sda`/`nvme0n1` visibles, signifie que le disque de l'hôte n'est qu'à un `mount` de distance.

## Sous-sujets

- **[Périphérique bloc de l'hôte](host-block-device.md)** : monter le système de fichiers racine de l'hôte et y effectuer chroot.
- **[Périphériques de mémoire du noyau](kernel-memory-devices.md)** : lire et écrire la RAM via /dev/mem et consorts.
- **[mknod](mknod.md)** : recréer un nœud de périphérique lorsque le chemin manque mais que la capacité est présente.

## Références

- [man 7 cgroups : device controller](https://man7.org/linux/man-pages/man7/cgroups.7.html)
- [Docker : --device and device cgroup rules](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)
- [Trail of Bits : Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
