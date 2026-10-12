---
title: "sysrq-trigger : piloter les actions magic-sysrq du noyau de l'hôte depuis un conteneur"
order: 3
description: "Écrire un caractère dans /proc/sysrq-trigger invoque le gestionnaire magic-sysrq du noyau sur l'hôte : synchroniser les disques, tuer des processus, dégeler les systèmes de fichiers, ou faire planter et redémarrer la machine. Un conteneur dont le procfs de l'hôte est monté en écriture l'utilise pour perturber l'hôte ou pour combiner la suppression de processus avec d'autres primitives, bien que sa valeur directe pour l'exécution de code soit limitée."
keywords:
  - sysrq-trigger
  - magic sysrq
  - proc sysrq
  - déni de service
  - évasion de conteneur
---

# sysrq-trigger

`/proc/sysrq-trigger` est un point d'entrée en écriture seule vers le gestionnaire magic-sysrq du noyau. Un seul caractère qui y est écrit exécute la commande sysrq correspondante contre le noyau de l'hôte : `s` synchronise les disques, `e` envoie SIGTERM à tous les processus sauf init, `i` envoie SIGKILL à tous, `f` invoque le tueur de manque de mémoire, `c` fait planter le noyau, et `b` redémarre immédiatement sans démonter. Ces actions agissent sur l'hôte car sysrq est global, de sorte qu'un conteneur dont le procfs de l'hôte est monté en écriture peut atteindre des effets à l'échelle de l'hôte.

Confirmez l'accès et que sysrq est activé :

```bash
[ -w /proc/sysrq-trigger ] && echo writable
cat /proc/sys/kernel/sysrq          # bitmask of allowed sysrq functions; 1 = all
```

## Actions disponibles

```bash
echo s > /proc/sysrq-trigger        # sync all mounted filesystems
echo e > /proc/sysrq-trigger        # SIGTERM to all user processes (mass kill)
echo i > /proc/sysrq-trigger        # SIGKILL to all user processes
echo c > /proc/sysrq-trigger        # crash the host (kernel panic)
echo b > /proc/sysrq-trigger        # immediate reboot, no clean unmount
```

## Notes d'exploitation

- Il s'agit principalement d'une primitive de déni de service et de perturbation, et non d'une évasion par exécution de code directe : elle ne peut pas exécuter un programme de l'attaquant comme le peuvent [core_pattern](core_pattern.md) ou [uevent_helper](uevent-helper.md).
- Les actions de suppression massive (`e`, `i`) peuvent être enchaînées : tuer un processus de l'hôte peut forcer un superviseur à le redémarrer d'une manière que l'attaquant influence, ou libérer un verrou dont l'attaquant a besoin, mais à soi seul cela ne fait que détruire la disponibilité.
- Le masque de bits `sysrq` dans `/proc/sys/kernel/sysrq` peut restreindre les fonctions autorisées ; `0` désactive entièrement sysrq et met la technique en échec.
- Y parvenir nécessite que le procfs de l'hôte soit monté en écriture, généralement un conteneur privilégié.

## Références

- [Documentation du noyau : touche magic SysRq](https://docs.kernel.org/admin-guide/sysrq.html)
- [HackTricks : sensitive mounts](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
