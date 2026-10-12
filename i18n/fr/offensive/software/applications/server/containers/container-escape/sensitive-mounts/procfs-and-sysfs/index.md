---
title: "procfs et sysfs : pseudo-fichiers noyau que l'hôte exécute en tant que root"
order: 3
description: "Plusieurs fichiers sous /proc/sys et /sys sont des chemins d'assistant en espace utilisateur ou des déclencheurs : le noyau de l'hôte les lit et exécute le programme référencé en tant que root dans les espaces de noms de l'hôte, ou expose la mémoire de l'hôte. Lorsqu'un conteneur a le véritable procfs ou sysfs de l'hôte monté en écriture, écrire dans core_pattern, modprobe ou uevent_helper, ou lire kcore, mène directement à l'hôte."
keywords:
  - procfs sysfs
  - usermode helper
  - core_pattern
  - uevent_helper
  - évasion de conteneur
---

# procfs et sysfs

`/proc` et `/sys` ne sont pas des fichiers ordinaires : de nombreuses entrées sont des points de contrôle que le noyau relit et sur lesquels il agit. Une poignée d'entre elles nomment un programme que le noyau exécute, en tant que root dans les espaces de noms init de l'hôte, lorsqu'un événement survient : un plantage (`core_pattern`), un chargement automatique de module noyau (`modprobe`), ou un uevent de périphérique (`uevent_helper`). D'autres exposent directement la mémoire de l'hôte (`kcore`) ou permettent à un processus de solliciter le noyau de l'hôte (`sysrq-trigger`). Il ne s'agit d'évasions que lorsque le conteneur a le véritable procfs ou sysfs de l'hôte monté et accessible en écriture, ce qui se produit avec un `-v /proc:/host/proc` imprudent, un remontage de procfs, ou un conteneur privilégié.

Vérifiez ce qui est exposé et accessible en écriture :

```bash
mount | grep -E 'proc|sys'                       # look for host proc/sys without "ro"
ls -l /proc/sys/kernel/core_pattern /proc/sys/kernel/modprobe 2>/dev/null
ls -l /sys/kernel/uevent_helper 2>/dev/null
for f in /proc/sys/kernel/core_pattern /proc/sys/kernel/modprobe /sys/kernel/uevent_helper; do
  [ -w "$f" ] && echo "writable: $f"; done
```

## Sous-thèmes

- **[core_pattern](core_pattern.md)** : rediriger par pipe le core dump d'un processus qui plante vers un programme de l'hôte.
- **[chemin modprobe](modprobe-path.md)** : détourner le chargeur automatique de modules que le noyau exécute en tant que root.
- **[uevent_helper](uevent-helper.md)** : exécuter un programme sur un uevent de périphérique.
- **[binfmt_misc](binfmt-misc.md)** : enregistrer un interpréteur pour un format de fichier.
- **[sysrq-trigger](sysrq-trigger.md)** : invoquer les actions magic-sysrq du noyau de l'hôte.
- **[accès aux processus de l'hôte](host-process-access.md)** : lire la mémoire et l'environnement des processus de l'hôte.
- **[lecture mémoire kcore](kcore-memory-read.md)** : extraire la mémoire du noyau de l'hôte via /proc/kcore.

## Références

- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
- [HackTricks : sensitive mounts](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
- [Documentation du noyau : interfaces call_usermodehelper](https://docs.kernel.org/admin-guide/sysctl/kernel.html)
