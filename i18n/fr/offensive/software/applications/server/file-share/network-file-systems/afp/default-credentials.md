---
order: 2
title: "Identifiants par défaut : comptes faibles et constructeur sur les appliances AFP"
description: "AFP se rencontre le plus souvent sur des appliances NAS livrées avec des comptes administratifs par défaut et des politiques de mots de passe faibles. Essayer les identifiants constructeur par défaut et faire du spray de mots de passe courants contre l'authentification AFP, ou l'administration web de l'appliance, donne un accès authentifié aux partages et fréquemment l'administration de l'appareil, puisque les mêmes comptes couvrent les deux."
keywords:
  - identifiants par défaut
  - appliance nas
  - afp
  - password spray
  - netatalk
---

# Identifiants par défaut

Le foyer habituel d'AFP est une appliance NAS (et quelques serveurs Linux/Netatalk), et les appliances sont livrées avec des comptes par défaut et des politiques de mots de passe laxistes. Quand l'accès invité est désactivé, le coup suivant est les identifiants : les valeurs par défaut du constructeur et les mots de passe courants, essayés directement contre l'authentification AFP et contre l'administration web de l'appliance, qui partage généralement le même magasin d'utilisateurs. Un succès donne un accès authentifié aux partages et souvent l'administration de l'appareil, puisque le compte admin couvre à la fois l'accès aux fichiers et la gestion.

```bash
# enumerate the auth modules offered (which UAMs/credentials are accepted)
nmap -p548 --script afp-serverinfo <target>
# try credentials against AFP (afp-brute, or a client login)
nmap -p548 --script afp-brute --script-args userdb=users.txt,passdb=pass.txt <target>
# the appliance web admin frequently shares the account store
#   try vendor defaults: admin/admin, admin/<blank>, admin/password, root/<vendor>
curl -sk https://<target>/   # identify the NAS vendor/model to look up its defaults
```

## Notes d'exploitation

- Identifiez d'abord le constructeur et le modèle de l'appliance (bannière de l'interface web, nom de serveur `afp-serverinfo`) ; les identifiants par défaut sont propres au constructeur et bien documentés.
- Le magasin d'utilisateurs AFP et le magasin d'utilisateurs de l'administration web sont généralement les mêmes sur un NAS, de sorte qu'un identifiant qui fonctionne pour l'un fonctionne généralement pour l'autre, transformant l'accès aux fichiers en administration de l'appareil (et vice versa).
- Restez prudent avec le spray face à toute politique de verrouillage ; les appareils NAS varient, et certains n'ont aucun verrouillage du tout, ce qui favorise une liste de mots de passe plus large.
- L'administration de l'appareil sur un NAS permet souvent d'activer SSH, d'ajouter des utilisateurs ou de lire tous les partages, faisant passer un point d'appui sur un partage de fichiers à un contrôle complet de l'appliance.

## Références

- [Netatalk authentication (UAMs)](https://netatalk.io/docs)
- [nmap afp-brute](https://nmap.org/nsedoc/scripts/afp-brute.html)
