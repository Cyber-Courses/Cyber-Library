---
title: "Évasion de conteneur : sortir d'un conteneur vers l'hôte"
order: 1
description: "Évasion de conteneur indépendante du runtime : sortir vers l'hôte via une configuration trop permissive (le flag privileged, des capacités dangereuses, l'accès aux périphériques, un confinement faible), des bind mounts dangereux, des espaces de noms hôte partagés, des exploits de runtime et de noyau, et des évasions depuis des runtimes bac à sable."
keywords:
  - évasion de conteneur
  - container breakout
  - conteneur privilégié
  - espace de noms hôte
  - exploit de runtime de conteneur
---

# Évasion de conteneur

Une évasion de conteneur transforme l'exécution de code à l'intérieur d'un conteneur en exécution de code sur l'hôte. Elle ne dépend presque jamais du moteur : les mêmes primitives fonctionnent depuis Docker, Podman, containerd et un pod Kubernetes, car tous s'appuient sur les mêmes fonctionnalités du noyau. Ce qui diffère, c'est seulement la façon dont le paramètre dangereux a été accordé au conteneur, raison pour laquelle les couches de runtime et d'orchestration pointent ici au lieu de redocumenter l'évasion.

## Triage d'abord

Avant d'essayer la moindre voie, identifiez ce qui a été accordé au conteneur ; la sortie détermine quel sous-sujet s'applique et épargne des tentatives inutiles :

```bash
# Capabilities, seccomp, and LSM confinement
grep -E 'CapEff|Seccomp' /proc/self/status
capsh --decode=$(grep CapEff /proc/self/status | awk '{print $2}')
cat /proc/self/attr/current 2>/dev/null         # AppArmor profile, or "unconfined"

# Is container root real host root?
cat /proc/self/uid_map                           # "0 0 4294967295" => host root

# Host paths, sockets, and sensitive kernel files mounted in
findmnt -o TARGET,SOURCE,OPTIONS | grep -vE 'overlay|proc|sysfs|tmpfs|cgroup'
ls -l /var/run/docker.sock /run/containerd/containerd.sock 2>/dev/null
for f in /proc/sys/kernel/core_pattern /proc/sys/kernel/modprobe /sys/kernel/uevent_helper; do
  [ -w "$f" ] && echo "writable: $f"; done

# Shared host namespaces
ls /proc | grep -qE '^[0-9]+$' && readlink /proc/1/exe   # host init => shared PID ns
ip addr | grep -q 'docker0\|host' && ss -tlnp 2>/dev/null | head

# Devices and runtime versions (for the exploit routes)
ls -l /dev | grep -E 'sd|nvme|mem' ; runc --version 2>/dev/null; uname -r
```

Lisez-la dans l'ordre : un `CapEff` complet avec `Seccomp: 0` indique `--privileged` ; une seule capacité ajoutée pointe vers une page [Abus de capacité](privileged-configuration/capability-abuse/index.md) ; un bind mount hôte ou un socket est un [Montage sensible](sensitive-mounts/index.md) ; des espaces de noms partagés mènent aux [Espaces de noms hôte partagés](shared-host-namespaces/index.md) ; et un conteneur verrouillé n'ayant rien de tout cela ne laisse que les [Exploits de runtime et de noyau](runtime-and-kernel-exploits/index.md). Un `uid_map` mappé avertit que les voies par capacité ne donneront peut-être qu'une identité remappée, et non le root de l'hôte.

Les primitives se regroupent selon ce qui rend l'évasion possible.

## Sous-sujets

- **[Configuration privilégiée](privileged-configuration/index.md)** : trop de privilèges ont été accordés au conteneur, du flag privileged tout-en-un à une seule capacité dangereuse, à l'accès brut aux périphériques, ou à un profil seccomp ou AppArmor désactivé.
- **[Montages sensibles](sensitive-mounts/index.md)** : un chemin de l'hôte, un socket de contrôle du runtime, ou le procfs et le sysfs de l'hôte ont été bind-montés dans le conteneur.
- **[Espaces de noms hôte partagés](shared-host-namespaces/index.md)** : le conteneur partage l'espace de noms PID, réseau, IPC ou utilisateur de l'hôte.
- **[Exploits de runtime et de noyau](runtime-and-kernel-exploits/index.md)** : une faille d'exécution de code dans le runtime OCI ou le shim, ou dans le noyau hôte partagé.
- **[Évasions de runtime bac à sable](sandboxed-runtime-escapes/index.md)** : sortir d'un bac à sable qui s'interpose sur le noyau, comme gVisor ou un runtime microVM.

## Références

- [Trail of Bits : Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [NCC Group : Understanding and Hardening Linux Containers](https://research.nccgroup.com/2016/04/13/understanding-and-hardening-linux-containers/)
- [man 7 namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)
