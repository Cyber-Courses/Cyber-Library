---
title: "mknod : recréer un nœud de périphérique de l'hôte à l'intérieur du conteneur"
description: "Même lorsque le /dev d'un conteneur ne liste pas un périphérique sensible, un processus détenant CAP_MKNOD et connaissant les numéros majeur et mineur du périphérique peut recréer le nœud avec mknod puis l'utiliser. Cela reconstruit l'accès au disque de l'hôte ou à la mémoire brute lorsque le cgroup de périphériques autorise encore le périphérique mais que le nœud n'a pas été créé."
keywords:
  - mknod
  - cap_mknod
  - majeur mineur de périphérique
  - nœud de périphérique
  - évasion de conteneur
---

# mknod

Un nœud de périphérique ne porte aucune donnée propre : ce n'est qu'une entrée de système de fichiers étiquetant un numéro majeur et un numéro mineur que le noyau résout en un pilote. `mknod` crée une telle entrée. Si le conteneur détient `CAP_MKNOD` (membre de l'ensemble par défaut de Docker) et que le cgroup de périphériques autorise encore un périphérique, un attaquant peut recréer le nœud du disque de l'hôte ou d'un périphérique de mémoire brute même si le `/dev` de l'image ne l'incluait pas, puis utiliser ce nœud pour atteindre l'hôte.

Vérifiez la capacité et ce que le cgroup de périphériques autorise :

```bash
capsh --print | grep -o cap_mknod
cat /sys/fs/cgroup/devices/devices.list 2>/dev/null
# "b *:* rwm" allows all block devices; "c 1:1 rwm" would allow /dev/mem
```

Le `devices.list` du cgroup de périphériques est le véritable verrou : `mknod` réussit à créer le nœud quoi qu'il arrive, mais son ouverture est refusée à moins que le cgroup n'autorise ce majeur/mineur. Une liste permissive (courante sous `--privileged`, qui positionne `a *:* rwm`) signifie que tout nœud recréé fonctionne.

## Recréer et utiliser un périphérique bloc

```bash
# host root disk is typically major 8 (sd*) or 259 (nvme); minor from /proc/partitions
cat /proc/partitions                     # find major:minor of the host root partition
mknod /tmp/hostdisk b 8 1                # recreate /dev/sda1 as a block node
mount /tmp/hostdisk /mnt/host 2>/dev/null || \
  debugfs -R 'cat /etc/shadow' /tmp/hostdisk   # read without mounting
```

Pour un périphérique de mémoire brute :

```bash
mknod /tmp/mem c 1 1                      # /dev/mem is char major 1 minor 1
```

## Notes d'exploitation

- Que `mknod` crée le nœud est nécessaire mais pas suffisant ; la liste d'autorisation du cgroup de périphériques décide si son ouverture est permise. Lisez d'abord `devices.list` pour savoir quels majeurs/mineurs sont utilisables.
- Découvrez les numéros majeur/mineur depuis `/proc/partitions` pour les périphériques bloc et depuis les conventions du `Documentation/admin-guide/devices.txt` du noyau pour les périphériques caractère (mem est 1:1, port est 1:4).
- C'est l'élément déclencheur lorsqu'une voie de périphérique est théoriquement ouverte (cgroup permissif) mais que le nœud manque ; combinez avec [Périphérique bloc de l'hôte](host-block-device.md) ou [Périphériques de mémoire du noyau](kernel-memory-devices.md) pour la lecture/écriture effective.

## Références

- [man 2 mknod](https://man7.org/linux/man-pages/man2/mknod.2.html)
- [Linux devices.txt (attributions majeur/mineur)](https://www.kernel.org/doc/Documentation/admin-guide/devices.txt)
- [HackTricks : CAP_MKNOD](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_mknod)
