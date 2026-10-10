---
title: "rsync : attaquer le démon rsync"
description: "rsync s'exécute en tant que démon sur le port TCP 873, exposant des modules nommés (partages), fréquemment avec un accès anonyme et sans authentification. La surface offensive comprend l'énumération des modules, la lecture anonyme de leur contenu, et l'abus de modules inscriptibles pour y déposer des fichiers, les privilèges du démon et tout chroot configuré déterminant jusqu'où une écriture porte."
keywords:
  - rsync
  - port 873
  - module
  - anonymous
  - rsync daemon
---

# rsync

rsync est surtout connu comme outil de synchronisation de fichiers via SSH, mais il s'exécute aussi en tant que démon autonome à l'écoute sur le port TCP 873, exposant des modules nommés qui se comportent comme des partages. Les modules du démon sont configurés dans `rsyncd.conf`, et ils sont couramment laissés anonymes (pas d'`auth users`) et parfois inscriptibles, avec des réglages de chroot et de privilèges variables. La surface offensive comprend l'énumération des modules exposés, la lecture de leur contenu sans identifiants, et l'écriture dans tout module qui l'autorise, ce qui dépose des fichiers sur l'hôte dans les limites de ce que le chroot et les privilèges du démon permettent.

```bash
# list modules exposed by the rsync daemon (no credentials needed to list)
rsync rsync://<target>/                       # or: rsync <target>::
nmap -p873 --script rsync-list-modules <target>
```

## Sous-thèmes

- **[Énumération des modules](module-enumeration.md)** : lister les modules et leur accès.
- **[Accès anonyme aux fichiers](anonymous-file-access.md)** : lire le contenu des modules sans authentification.
- **[Accès en écriture](write-access.md)** : déposer des fichiers via un module inscriptible.

## Références

- [rsyncd.conf manual](https://download.samba.org/pub/rsync/rsyncd.conf.5)
- [rsync manual](https://download.samba.org/pub/rsync/rsync.1)
- [HackTricks : rsync (873)](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)
