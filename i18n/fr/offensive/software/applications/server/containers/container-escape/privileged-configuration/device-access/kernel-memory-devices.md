---
title: "Périphériques de mémoire du noyau : patcher le noyau en cours d'exécution via /dev/mem"
description: "Les conteneurs qui exposent les nœuds de périphérique de mémoire brute /dev/mem, /dev/kmem ou /dev/port laissent un processus lire et écrire directement la mémoire physique. Un attaquant mappe des pages du noyau, localise une structure d'identifiants de processus ou un interrupteur de sécurité, et l'écrase, s'évadant de l'isolation du conteneur car la mémoire physique appartient à l'hôte, et non à un espace de noms."
keywords:
  - dev/mem
  - mémoire du noyau
  - mémoire physique
  - écrasement d'identifiants
  - évasion de conteneur
---

# Périphériques de mémoire du noyau

Les nœuds `/dev/mem` (mémoire physique), `/dev/kmem` (mémoire virtuelle du noyau) et `/dev/port` (ports d'E/S) sont des fenêtres sur le noyau hôte en direct. Ils ne sont pas cloisonnés par espace de noms, de sorte qu'une écriture via `/dev/mem` depuis l'intérieur d'un conteneur modifie la mémoire du noyau de l'hôte. Lorsqu'un conteneur expose ces nœuds, généralement sous `--privileged` ou via un `--device=/dev/mem` explicite, un attaquant qui y a accès peut patcher des structures de données du noyau pour élever ses privilèges ou désactiver des protections.

Confirmez les nœuds et si l'accès à la RAM est autorisé :

```bash
ls -l /dev/mem /dev/kmem /dev/port 2>/dev/null
grep -E 'STRICT_DEVMEM|IO_STRICT_DEVMEM' /boot/config-$(uname -r) 2>/dev/null
# STRICT_DEVMEM=y confines /dev/mem to device ranges and blocks general RAM
```

## Voie : écraser les identifiants d'un processus

L'objectif classique est de trouver la structure `cred` d'un processus shell et de mettre à zéro ses champs UID/GID, le transformant en root, ou d'effacer un drapeau d'application d'un LSM. L'attaquant a besoin d'une adresse noyau (depuis `/proc/kallsyms`, un décalage connu pour la version du noyau, ou une fuite) et du mappage physique pour `/dev/mem`.

```c
int fd = open("/dev/mem", O_RDWR);
off_t phys = virt_to_phys(target_kvaddr);          // via known offset / leak
void *m = mmap(NULL, 0x1000, PROT_READ|PROT_WRITE, MAP_SHARED, fd, phys & ~0xfff);
uint32_t *cred = (uint32_t *)((char*)m + (phys & 0xfff));
cred[0] = cred[1] = cred[2] = cred[3] = 0;          // uid,gid,suid,sgid -> 0
```

Lire d'abord avec le même mappage vous permet de confirmer la signature de la structure (par exemple l'UID attendu du processus cible) avant d'écrire, ce qui évite de corrompre la mauvaise page.

## Voie : ports d'E/S

`/dev/port` et la surface `iopl`/`ioperm` atteignent directement les contrôleurs de périphériques. Sur un matériel adapté, cela pilote un périphérique capable de DMA pour lire ou écrire une mémoire physique arbitraire, obtenant la même écriture du noyau sans accès à la RAM via `/dev/mem`.

## Notes d'exploitation

- `CONFIG_STRICT_DEVMEM` (par défaut sur les distributions modernes) bloque l'accès général à la RAM via `/dev/mem`, ne laissant que les plages de périphériques ; lorsqu'il est positionné, préférez [CAP_SYS_MODULE](../capability-abuse/cap-sys-module.md) ou un montage de disque. `/dev/kmem` est retiré de la plupart des noyaux actuels.
- Cette voie dépend de décalages de noyau exacts pour la version précise en cours d'exécution ; des décalages incorrects corrompent la mémoire et font paniquer l'hôte, alors identifiez d'abord précisément `uname -r` et le noyau de la distribution.
- L'accès à ces nœuds implique aussi la voie de capacité d'E/S brute ; voir [CAP_SYS_RAWIO](../capability-abuse/cap-sys-rawio.md).

## Références

- [man 4 mem](https://man7.org/linux/man-pages/man4/mem.4.html)
- [Kernel docs : /dev/mem and STRICT_DEVMEM](https://www.kernel.org/doc/html/latest/driver-api/device-io.html)
- [HackTricks : Docker breakout](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation)
