---
title: "Configuration et exposition : faiblesses de déploiement de Zabbix"
order: 2
description: "Indépendamment d'un exploit unique, Zabbix est souvent déployé de façon non sécurisée : le frontend web et l'API exposés à des réseaux non fiables, et des rôles et groupes d'utilisateurs trop permissifs qui accordent à des comptes ordinaires les droits de Super admin ou d'exécution de scripts qui mènent à l'exécution de code. Ces faiblesses de configuration élargissent qui peut atteindre la plateforme et ce qu'un compte donné peut faire."
keywords:
  - zabbix exposure
  - exposed frontend
  - roles
  - permissions
  - super admin
---

# Configuration et exposition

Au-delà des exploits spécifiques, la façon dont Zabbix est déployé détermine son degré d'exposition. Deux faiblesses de configuration reviennent. Le frontend et l'API sont fréquemment accessibles depuis des réseaux non fiables (Internet, ou un réseau interne plat), ce qui expose la connexion, la version et la surface invité/identifiants par défaut à n'importe qui. Et le modèle de rôles et de groupes d'utilisateurs est souvent trop permissif : des comptes se voient accorder le rôle de Super admin ou les permissions de script/item qui mènent à l'exécution de code alors qu'ils n'en ont pas besoin, de sorte que la compromission d'un compte ordinaire rapporte bien plus que ne le justifiait son usage. Ce ne sont pas des exploits ponctuels mais les conditions qui rendent les autres attaques accessibles et impactantes.

## Sous-thèmes

- **[Interface web exposée](exposed-web-interface.md)** : le frontend et l'API accessibles depuis des réseaux non fiables.
- **[Permissions faibles](weak-permissions.md)** : rôles et groupes d'utilisateurs surprivilégiés.

## Références

- [Zabbix : user roles and permissions](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/user_roles)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
