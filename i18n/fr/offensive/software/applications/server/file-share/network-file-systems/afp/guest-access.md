---
title: "Accès invité : accès non authentifié aux volumes AFP"
order: 3
description: "De nombreux serveurs AFP, en particulier les appliances NAS et les installations Netatalk, activent l'accès invité, annoncé sous la capacité No User Authent. Un client se connectant en tant qu'invité monte et parcourt les volumes exposés sans identifiants, lisant (et parfois écrivant) les données partagées, faisant de l'AFP avec invité activé un point d'ancrage immédiat avant identifiants."
keywords:
  - afp guest
  - no user authent
  - netatalk
  - non authentifié
  - accès aux volumes
---

# Accès invité

AFP prend en charge une connexion invité, et elle est couramment laissée activée : les équipements NAS sont livrés avec elle activée pour plus de commodité, et les configurations Netatalk l'autorisent fréquemment. Le serveur annonce la capacité dans sa réponse `afp-serverinfo` sous le UAM (User Authentication Module) `No User Authent`. Un client qui se connecte en tant qu'invité monte les volumes exposés et lit leur contenu, et là où le volume le permet, y écrit, le tout sans aucun identifiant. Cela fait d'un serveur AFP avec invité activé un point d'ancrage immédiat de reconnaissance et de butin.

```bash
# confirm guest/No-User-Authent is offered
nmap -p548 --script afp-serverinfo <target>    # look for "No User Authent" in UAMs
nmap -p548 --script afp-showmount,afp-ls <target>   # list volumes and contents as guest
# mount a volume as guest (macOS, or afpfs-ng on Linux)
mount_afp afp://;AUTH=No%20User%20Authent@<target>/<Volume> /mnt/afp   # macOS
afp_client mount -u guest <target> <Volume> /mnt/afp                    # afpfs-ng
```

## Notes d'exploitation

- La liste des UAM de `afp-serverinfo` est le signe révélateur : `No User Authent` signifie que la connexion invité est acceptée, testez-la donc avant toute autre chose.
- L'accès invité est en lecture au minimum et parfois en écriture, selon la configuration du volume ; parcourez à la recherche du même butin que n'importe quel partage de fichiers (configurations, clés, sauvegardes), voir l'approche de butin SMB pour les motifs de contenu.
- Les appliances NAS et les cibles Time Machine sont les cas courants avec invité activé ; les volumes Time Machine contiennent des sauvegardes système complètes qui valent la peine d'être récupérées.
- L'accès invité est avant identifiants ; combinez-le avec les [identifiants par défaut](default-credentials.md) si l'invité est désactivé mais que l'appliance utilise des comptes faibles.

## Outils

- [afpfs-ng (Linux AFP client)](https://sourceforge.net/projects/afpfs-ng/)
- [nmap AFP scripts](https://nmap.org/nsedoc/)

## Références

- [Netatalk configuration (afp.conf)](https://netatalk.io/docs)
- [nmap afp-serverinfo](https://nmap.org/nsedoc/scripts/afp-serverinfo.html)
