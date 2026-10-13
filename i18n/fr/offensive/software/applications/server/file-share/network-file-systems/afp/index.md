---
title: "AFP : attaquer l'Apple Filing Protocol"
description: "AFP sert des partages de fichiers Apple sur TCP 548, nativement sur les anciennes versions de macOS et via Netatalk sur Linux et les équipements NAS. La surface offensive est l'accès invité que de nombreux serveurs laissent activé, les identifiants par défaut et faibles sur les appliances, et l'historique de l'implémentation Netatalk en matière de vulnérabilités sérieuses d'exécution de code à distance pré-authentification."
keywords:
  - afp
  - apple filing protocol
  - netatalk
  - port 548
  - accès invité
---

# AFP

AFP (Apple Filing Protocol) sert des partages de fichiers de style Apple sur TCP 548. C'était le partage de fichiers natif de macOS avant que SMB ne prenne le relais, et il persiste sur les anciens serveurs macOS, les cibles Time Machine et surtout sur les appliances NAS et les serveurs Linux faisant tourner Netatalk, l'implémentation AFP open-source. D'un point de vue offensif, il y a trois fils : l'accès invité, que de nombreux serveurs AFP activent par défaut et qui expose les volumes sans identifiants ; les identifiants par défaut et faibles sur les appliances ; et l'implémentation de Netatalk, qui a porté plusieurs bugs sérieux d'exécution de code à distance pré-authentification qui font d'un Netatalk exposé une cible de compromission directe.

```bash
# discover AFP and enumerate server info/volumes
nmap -p548 --script afp-serverinfo,afp-showmount,afp-ls <target>
# afp-serverinfo reveals the server name, version, and supported auth (incl. "No User Authent" = guest)
```

## Sous-sujets

- **[Accès invité](guest-access.md)** : accès non authentifié aux volumes AFP.
- **[Identifiants par défaut](default-credentials.md)** : comptes faibles et par défaut du fabricant sur les appliances.
- **[Exploits Netatalk](netatalk-exploits.md)** : l'historique de RCE pré-auth de l'implémentation.

## Références

- [Netatalk project](https://netatalk.io/)
- [nmap AFP scripts](https://nmap.org/nsedoc/)
- [Apple Filing Protocol reference](https://developer.apple.com/library/archive/documentation/Networking/Conceptual/AFP/)
