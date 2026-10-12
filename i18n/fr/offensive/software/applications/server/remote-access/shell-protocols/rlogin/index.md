---
title: "rlogin : attaquer le service de connexion distante"
order: 4
description: "rlogin (rlogind sur TCP 513) ouvre une session de connexion interactive, en s'authentifiant par la même confiance d'hôte .rhosts/hosts.equiv que rsh, ou en se repliant sur un mot de passe en clair. Les attaques sont la connexion sans mot de passe par abus de confiance, la capture du mot de passe et de la session en clair, l'énumération de noms d'utilisateur à partir du comportement de connexion, et le détournement de session."
keywords:
  - rlogin
  - rlogind
  - port 513
  - rhosts
  - cleartext
---

# rlogin

`rlogin` ouvre une session de connexion interactive sur un hôte distant via `rlogind` sur TCP 513. Il s'authentifie de la même manière que rsh, confiance basée sur l'hôte via `~/.rhosts` et `/etc/hosts.equiv`, et se replie sur une invite de mot de passe en clair lorsque la confiance ne s'applique pas. Les attaques suivent : connexion sans mot de passe en abusant de ou en implantant la confiance, capture du mot de passe en clair et de toute la session lorsqu'un mot de passe est utilisé, énumération de noms d'utilisateur à partir de la façon dont la connexion répond, et détournement de la session en clair en direct. Parce qu'il accorde un shell interactif, un rlogin réussi est un point d'ancrage direct.

```bash
# trusted login (no password) or password login
rlogin -l <user> <target>
nmap -p513 -sV <target>
```

## Sous-thèmes

- **[Contournement rhosts](rhosts-bypass.md)** : connexion sans mot de passe via la confiance d'hôte.
- **[Mots de passe en clair](cleartext-passwords.md)** : capturer le mot de passe quand la confiance n'est pas utilisée.
- **[Énumération de noms d'utilisateur](username-enumeration.md)** : découvrir des utilisateurs valides à partir du comportement de connexion.
- **[Détournement de session](session-hijacking.md)** : prendre le contrôle de la session rlogin en direct.

## Références

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rlogin (513)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
