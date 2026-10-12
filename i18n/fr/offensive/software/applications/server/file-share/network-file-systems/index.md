---
title: "Systèmes de fichiers réseau : attaquer les protocoles de système de fichiers partagé"
description: "Les systèmes de fichiers réseau exportent des systèmes de fichiers sur le réseau pour montage par les clients : SMB sur Windows, NFS sur Unix et AFP sur les équipements Apple et NAS. Chacun fait confiance au client à des degrés divers, SMB via les sessions et NTLM, NFS via les UID affirmés par le client, AFP via les comptes invité et d'appliance, et chacun expose l'énumération, l'abus d'accès, le butin et les failles d'implémentation."
keywords:
  - système de fichiers réseau
  - smb
  - nfs
  - afp
  - partage de fichiers
---

# Systèmes de fichiers réseau

Un système de fichiers réseau exporte le système de fichiers d'un serveur pour que les clients puissent le monter et l'utiliser comme s'il était local. Les trois qui comptent sont SMB (le standard Windows, aussi Samba), NFS (le standard Unix) et AFP (le protocole d'Apple, désormais surtout sur les équipements NAS via Netatalk). Ils diffèrent fortement par le degré de confiance accordé au client : SMB établit des sessions et utilise NTLM/Kerberos, sa surface inclut donc le relais et l'abus de session ; NFS avec AUTH_SYS fait confiance à l'UID affirmé par le client, l'usurpation d'identité est donc triviale ; AFP s'appuie sur l'accès invité et les comptes d'appliance. À travers les trois, les mouvements récurrents sont l'énumération des exports et des accès, l'abus des modèles d'accès faibles, le butinage des données lisibles et l'exploitation de l'implémentation du serveur.

## Sous-sujets

- **[SMB](smb/index.md)** : le protocole de partage de fichiers de Windows, sessions, partages, relais, butin et empoisonnement.
- **[NFS](nfs/index.md)** : le système de fichiers réseau Unix et son modèle de confiance AUTH_SYS.
- **[AFP](afp/index.md)** : l'Apple Filing Protocol et Netatalk.

## Références

- [MS-SMB2](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/)
- [NFS (nfs(5), RFC 8881)](https://man7.org/linux/man-pages/man5/nfs.5.html)
- [Netatalk / AFP](https://netatalk.io/)
