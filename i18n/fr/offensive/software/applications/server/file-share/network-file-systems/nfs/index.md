---
title: "NFS : attaquer le système de fichiers réseau Unix"
description: "NFS exporte des systèmes de fichiers Unix sur le réseau, et avec la sécurité traditionnelle AUTH_SYS il fait entièrement confiance aux identifiants d'utilisateur et de groupe fournis par le client. La surface offensive est l'énumération des exports et de leurs options, l'abus de no_root_squash pour écrire des fichiers en tant que root, l'usurpation d'UID et de GID pour lire et écrire les fichiers de n'importe quel utilisateur, et les points plus faibles de NFSv4 avec Kerberos."
keywords:
  - nfs
  - auth_sys
  - no_root_squash
  - exports
  - showmount
---

# NFS

NFS (Network File System) exporte des répertoires Unix sur le réseau. Sa faiblesse déterminante est le modèle de sécurité traditionnel `AUTH_SYS` (`sec=sys`), selon lequel le serveur fait confiance à n'importe quels identifiants d'utilisateur et de groupe que le client inclut dans chaque requête : il n'y a aucune authentification de l'identité revendiquée par le client, seulement les identifiants qu'il affirme. Cela fait d'un export NFS avec les options par défaut une porte quasi ouverte. La surface est l'énumération des exports et de leurs options, l'abus de `no_root_squash` pour agir en tant que root sur l'export, l'usurpation d'UID/GID arbitraires pour accéder aux fichiers de n'importe quel utilisateur, et les configurations comparativement plus difficiles de NFSv4-avec-Kerberos.

```bash
# discover exports and the RPC services behind NFS
showmount -e <target>                         # exported paths and allowed clients
rpcinfo -p <target>                           # mountd/nfs/portmapper ports
nmap -p111,2049 --script nfs-showmount,nfs-ls <target>
```

## Sous-sujets

- **[Énumération](enumeration.md)** : exports, options et montage.
- **[Abus de no_root_squash](no-root-squash-abuse.md)** : écrire des fichiers en tant que root sur un export.
- **[Usurpation d'UID et de GID](uid-and-gid-spoofing.md)** : se faire passer pour n'importe quel utilisateur sous AUTH_SYS.
- **[NFSv4 et Kerberos](nfsv4-and-kerberos.md)** : la configuration plus robuste et ses points faibles.

## Références

- [Linux exports(5) and NFS security](https://man7.org/linux/man-pages/man5/exports.5.html)
- [RFC 1813 (NFSv3) and RFC 8881 (NFSv4)](https://datatracker.ietf.org/doc/html/rfc8881)
- [HackTricks: NFS (2049)](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)
