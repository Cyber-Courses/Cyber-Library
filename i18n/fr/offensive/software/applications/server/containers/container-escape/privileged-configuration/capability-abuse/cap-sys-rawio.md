---
title: "CAP_SYS_RAWIO : patcher le noyau via l'accès brut aux ports et à la mémoire physique"
order: 6
description: "CAP_SYS_RAWIO octroie ioperm, iopl, et l'accès à /dev/mem, /dev/kmem et /dev/port. À l'intérieur d'un conteneur qui expose aussi ces nœuds de périphérique, elle laisse un attaquant lire et écrire directement la mémoire physique, localiser des structures du noyau et les écraser pour désactiver des protections ou élever un processus, contournant entièrement l'isolation par espaces de noms."
keywords:
  - cap_sys_rawio
  - dev/mem
  - mémoire physique
  - ioperm iopl
  - évasion de conteneur
---

# CAP_SYS_RAWIO

`CAP_SYS_RAWIO` autorise les entrées/sorties matérielles brutes : les appels système `ioperm(2)` et `iopl(2)` pour l'accès aux ports x86, et l'ouverture/lecture/écriture sur les périphériques de mémoire brute `/dev/mem`, `/dev/kmem` et `/dev/port` là où ils existent et ne sont pas bloqués par `CONFIG_STRICT_DEVMEM`. Comme la mémoire physique et les ports d'E/S sont une propriété de l'unique machine hôte, et non d'un quelconque espace de noms, y écrire atteint directement la mémoire du noyau et ignore les frontières du conteneur.

Confirmez la capacité et qu'un nœud de périphérique utilisable est présent :

```bash
capsh --print | grep -o cap_sys_rawio
grep CapEff /proc/self/status          # bit 17 set
ls -l /dev/mem /dev/port 2>/dev/null   # need a raw memory device exposed in the container
```

## Voie : écrire la mémoire du noyau via /dev/mem

Avec `/dev/mem` accessible en lecture et en écriture, l'attaquant mappe des pages physiques, cherche une structure connue du noyau et la patche. Un objectif courant est de localiser les identifiants d'un processus et d'écraser ses champs UID/GID à 0, ou d'inverser un drapeau de sécurité tel qu'un interrupteur d'application d'un LSM.

```c
int fd = open("/dev/mem", O_RDWR);
// mmap a window of physical memory and scan for a signature
void *p = mmap(0, LEN, PROT_READ|PROT_WRITE, MAP_SHARED, fd, phys_off);
// locate task_struct->cred, overwrite uid/gid/euid/egid with 0
```

Traduire une adresse virtuelle du noyau en un décalage physique pour `/dev/mem` nécessite de fuiter la base du noyau (par exemple depuis `/proc/kallsyms` s'il est lisible, ou un décalage connu pour le noyau en cours). Lorsque `/dev/mem` est restreint au premier 1 Mio par `STRICT_DEVMEM`, les voies des ports et de `iopl` demeurent pour le matériel qui mappe des registres de contrôle dans l'espace d'E/S.

## Voie : accès aux ports d'E/S

`iopl(3)` élève le niveau de privilège d'E/S du processus pour que les instructions `in`/`out` sur n'importe quel port réussissent, et `ioperm` active des ports spécifiques. Cela atteint directement les contrôleurs de périphériques et, sur un matériel adapté, les périphériques capables de DMA, qui peuvent être pilotés pour lire ou écrire une mémoire physique arbitraire.

## Notes d'exploitation

- Les noyaux modernes sont compilés avec `CONFIG_STRICT_DEVMEM` et souvent `CONFIG_IO_STRICT_DEVMEM`, qui confinent `/dev/mem` aux plages de périphériques et bloquent l'accès à la RAM ; vérifiez `zcat /proc/config.gz | grep DEVMEM` ou `/boot/config-$(uname -r)`. Lorsque la RAM est bloquée, cette capacité perd l'essentiel de sa valeur d'évasion.
- Cette voie est sensible au matériel et aux décalages et vaut mieux être réservée aux cas où des capacités plus propres ([CAP_SYS_MODULE](cap-sys-module.md), [CAP_SYS_ADMIN](cap-sys-admin.md)) sont absentes mais où un périphérique de mémoire brute est exposé.
- Des périphériques de mémoire brute exposés signifient généralement un conteneur `--privileged` ou un `--device=/dev/mem` explicite ; voir [Périphériques de mémoire du noyau](../device-access/kernel-memory-devices.md).

## Références

- [man 2 iopl](https://man7.org/linux/man-pages/man2/iopl.2.html)
- [man 4 mem](https://man7.org/linux/man-pages/man4/mem.4.html)
- [HackTricks : CAP_SYS_RAWIO](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_sys_rawio)
