---
order: 3
title: "Énumération de noms d'utilisateur : découvrir les comptes rlogin valides"
description: "Le flux de connexion de rlogin peut révéler quels noms d'utilisateur sont valides : le traitement par le service des comptes de confiance par rapport à ceux sans confiance, et des comptes existants par rapport à ceux inexistants, diffère, si bien que le comportement de l'invite distingue les utilisateurs réels. Une liste d'utilisateurs validée concentre l'abus de confiance et les attaques de mot de passe sur les comptes qui existent."
keywords:
  - username enumeration
  - rlogin
  - login prompt
  - valid user
  - trust
---

# Énumération de noms d'utilisateur

rlogin peut laisser fuir quels noms d'utilisateur existent par des différences dans son traitement de connexion. Selon l'implémentation, un utilisateur de confiance mais inexistant, un utilisateur existant sans confiance (invité à saisir un mot de passe) et un utilisateur inexistant (rejeté différemment) produisent des réponses distinguables, si bien que sonder le comportement de `-l <user>` sépare les comptes réels des invalides. Une liste d'utilisateurs validée concentre ensuite l'abus de confiance et les attaques de mot de passe, vous plantez des `.rhosts` pour, ou faites du spray/de la capture contre, des comptes qui existent réellement, et révèle quels comptes sont déjà de confiance (connexion sans mot de passe).

```bash
# probe usernames and classify the response
for u in root admin oracle bin operator; do
  echo "== $u =="; rlogin -l "$u" <target> </dev/null 2>&1 | head -2; done
# distinguish: immediate shell (trusted+exists), password prompt (exists, no trust),
#   rejection/error (invalid user) - the differences enumerate valid accounts
```

## Notes d'exploitation

- La distinction utile est à trois niveaux : un utilisateur qui se connecte sans mot de passe est à la fois valide et de confiance (victoire immédiate), un utilisateur invité à saisir un mot de passe est valide mais sans confiance (cible pour la capture/l'implantation), et un rejet différent marque un utilisateur invalide.
- Semez des candidats à partir des conventions des systèmes hérités (`root`, `oracle`, `operator`, `bin`, comptes applicatifs) et de toute autre énumération, puis validez ici.
- Un compte déjà de confiance découvert est une connexion directe sans mot de passe, donc l'énumération de noms d'utilisateur peut faire émerger le chemin le plus facile, pas seulement une liste de cibles.
- Alimentez les utilisateurs validés dans le [contournement rhosts](rhosts-bypass.md) (implantation de confiance) et la capture de [mot de passe en clair](cleartext-passwords.md).

## Références

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rlogin](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
