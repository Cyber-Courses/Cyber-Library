---
title: "Énumération d'utilisateurs : découverte de noms d'utilisateur SSH valides"
description: "SSH peut révéler quels noms d'utilisateur sont valides via des différences dans les méthodes d'authentification acceptées par utilisateur, et, sur certaines versions d'OpenSSH, via un canal auxiliaire temporel où les utilisateurs valides mettent mesurablement plus de temps à être rejetés. Une liste d'utilisateurs validée focalise le password spraying et les attaques par clé sur de vrais comptes et évite le bruit contre les comptes inexistants."
keywords:
  - énumération d'utilisateurs ssh
  - méthodes d'auth
  - attaque temporelle
  - openssh
  - nom d'utilisateur
---

# Énumération d'utilisateurs

Savoir quels noms d'utilisateur existent focalise les attaques d'identifiants et évite les tentatives gaspillées et bruyantes contre des comptes absents. SSH divulgue cela de deux façons. D'abord, les méthodes d'authentification que le serveur propose peuvent différer par utilisateur, un utilisateur configuré pour les clés uniquement contre un autorisant les mots de passe, ou un compte existant contre un inexistant, ce qu'une sonde des méthodes acceptées distingue. Ensuite, certaines versions d'OpenSSH présentaient une vulnérabilité temporelle : le serveur effectuait un vrai hachage (lent) de mot de passe pour les utilisateurs valides mais abandonnait tôt pour les invalides, donc le timing de la réponse révélait l'existence du compte.

```bash
# method-difference probe: compare accepted auth methods across candidate users
for u in root admin deploy svc backup; do
  echo -n "$u: "; ssh -o PreferredAuthentications=none -o StrictHostKeyChecking=no "$u@<target>" 2>&1 \
    | grep -i 'authentications that can continue'; done
# timing-based enumeration on vulnerable OpenSSH versions
nmap -p22 --script ssh-brute --script-args userdb=users.txt <target>   # (careful: attempts auth)
# dedicated tools implement the timing/version-specific checks
```

## Notes d'exploitation

- Le signal de différence de méthode est indépendant de la version et peu bruyant là où il existe : un utilisateur à qui est proposé `publickey` uniquement contre un autre à qui est proposé `publickey,password` distingue la configuration du compte et parfois son existence.
- Le canal auxiliaire temporel ne s'applique qu'à certaines versions d'OpenSSH ; faites correspondre la version de la [bannière](banner-grabbing.md) avant de vous y fier, et tenez compte de la gigue réseau en échantillonnant à plusieurs reprises.
- Amorcez les candidats à partir d'autres sources (RID cycling SMB, noms d'utilisateur d'applications web, email/OSINT, conventions de comptes de service) et validez-les ici avant la [force brute de mot de passe](../authentication/password-brute-force.md).
- L'énumération qui tente réellement une authentification est journalisée ; préférez la sonde de méthodes non authentifiante lorsque l'objectif est de rester discret.

## Outils

- [ssh-audit](https://github.com/jtesta/ssh-audit)
- [nmap ssh scripts](https://nmap.org/nsedoc/)

## Références

- [OpenSSH user enumeration advisories](https://www.openssh.com/security.html)
- [HackTricks: SSH user enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
