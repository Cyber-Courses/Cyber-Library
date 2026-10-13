---
order: 2
title: "Détournement de session : prendre le contrôle d'une session rlogin en direct"
description: "Une session rlogin est un flux TCP non chiffré authentifié uniquement à l'établissement, si bien qu'un attaquant on-path y injecte des commandes ou en prend le contrôle, s'exécutant en tant qu'utilisateur connecté. C'est l'attaque classique de détournement de session TCP des r-commands/Telnet, contournant l'authentification en chevauchant une session établie."
keywords:
  - session hijacking
  - rlogin
  - tcp injection
  - hunt
  - juggernaut
---

# Détournement de session

Une fois une session rlogin établie, ce n'est qu'un flux TCP non chiffré sans authentification par paquet, si bien qu'un attaquant on-path la détourne. En lisant le flux en clair pour apprendre l'état des numéros de séquence et d'acquittement TCP, l'attaquant injecte des segments portant des commandes que le serveur exécute en tant qu'utilisateur connecté, ou désynchronise le client légitime et prend entièrement le contrôle de la session. Cela contourne complètement l'authentification, l'attaquant n'a jamais besoin de confiance ni de mot de passe, il s'empare d'une session qu'un autre utilisateur a authentifiée, s'exécutant avec les privilèges de cet utilisateur. rlogin était une cible canonique des outils classiques de détournement de session.

```bash
# on-path position to the rlogin session (ARP/route); the cleartext stream reveals
# the TCP seq/ack state. Inject a command segment the server runs as the user:
#   e.g. append trust for durable re-entry
#   echo "+ +" >> ~/.rhosts
# classic automated tools (Hunt, Juggernaut) implemented rlogin/telnet TCP hijacking;
# modern exploitation crafts the segments directly from the observed seq/ack.
```

## Notes d'exploitation

- Injecter une seule commande (planter `.rhosts`, ajouter un utilisateur, lancer un reverse shell) est généralement l'objectif ; une prise de contrôle complète désynchronise et peut alerter l'utilisateur en direct, tandis qu'une injection discrète établit un accès durable.
- La commande injectée s'exécute en tant qu'utilisateur connecté, souvent administratif sur l'Unix hérité, donc une seule commande peut être une compromission totale.
- Elle ne nécessite qu'une position on-path et le flux en clair, aucun identifiant ni confiance ; c'est le pendant actif de la [capture passive de la session](cleartext-passwords.md).
- La technique identique s'applique à rsh et Telnet ; la conception partagée en clair et avec authentification uniquement à l'établissement en est la cause racine.

## Références

- [HackTricks: rlogin hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
