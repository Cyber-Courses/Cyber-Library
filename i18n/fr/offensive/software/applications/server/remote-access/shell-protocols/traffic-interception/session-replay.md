---
title: "Rejeu de session : rejeu des sessions r-command capturées"
description: "Parce que les r-commands n'authentifient qu'à l'établissement et transportent les sessions en clair, un attaquant qui capture le trafic peut détourner la session TCP en direct pour injecter des commandes au nom de l'utilisateur authentifié, et dans certains cas rejouer des échanges capturés. Cela exécute des commandes sans vaincre l'authentification, en chevauchant une session établie par quelqu'un d'autre."
keywords:
  - rejeu de session
  - détournement tcp
  - injection de commande
  - rsh
  - rlogin
---

# Rejeu de session

Les r-commands n'authentifient que pendant l'établissement de la connexion, après quoi la session est un flux TCP en clair non authentifié. Un attaquant qui capture ce flux ou s'y trouve l'exploite de deux façons. La plus directe et fiable est le détournement de session en direct : lire le flux en clair pour connaître l'état de séquence/acquittement TCP et injecter des segments transportant des commandes que le serveur exécute au nom de l'utilisateur authentifié. La seconde est le rejeu d'échanges capturés lorsque le serveur ne l'empêche pas. Les deux exécutent des commandes sans vaincre l'authentification, en chevauchant une session établie par une autre partie, et une seule commande injectée peut implanter une confiance durable.

```bash
# live hijack: on-path, read seq/ack from the cleartext stream, inject a command
#   e.g. append trust so later access needs no interception
#   echo "+ +" >> ~/.rhosts
# classic tools (Hunt, Juggernaut) automated rsh/rlogin TCP hijacking; modern
# exploitation crafts the injection segments directly from the observed stream.
```

## Notes d'exploitation

- Le détournement en direct est la forme pratique : injecter une seule commande (implanter un `.rhosts`, ajouter un utilisateur, lancer un reverse shell) dans la session établie, s'exécutant au nom de l'utilisateur authentifié, souvent administratif sur les Unix hérités.
- Une injection discrète d'une seule commande est préférable à une prise de contrôle complète, qui désynchronise et peut alerter l'utilisateur.
- Cela ne nécessite qu'une position sur le chemin et le flux en clair, ni identifiant ni confiance ; c'est le pendant actif de l'[interception de commandes](command-interception.md) et la même technique que le [détournement de session rlogin](../rlogin/session-hijacking.md) et Telnet.
- La conception commune (texte en clair, authentification à l'établissement seulement) partagée par rsh/rlogin/rexec est la cause racine qui rend possibles toutes les interceptions et tous les rejeux.

## Références

- [HackTricks: r-commands hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
