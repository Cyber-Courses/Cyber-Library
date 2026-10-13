---
order: 1
title: "Interception en clair : capturer la session rsh non chiffrée"
description: "rsh ne chiffre rien, si bien qu'un attaquant bien positionné capture les commandes exécutées, leur sortie et toute donnée de la session directement sur le fil. Là où une authentification de type mot de passe est utilisée plutôt que la confiance d'hôte, l'identifiant est capturé aussi, et la session peut être détournée comme avec les autres protocoles de shell distant en clair."
keywords:
  - cleartext
  - rsh
  - command capture
  - sniffing
  - interception
---

# Interception en clair

Comme les autres r-commands, `rsh` ne fournit aucun chiffrement, si bien qu'un attaquant sur le chemin lit tout l'échange : la commande exécutée, sa sortie complète et toute donnée transférée. Parce que rsh repose couramment sur la confiance basée sur l'hôte plutôt que sur des mots de passe, le butin est généralement la commande et la sortie (qui incluent souvent des données sensibles ou des identifiants secondaires que la commande manipule) plutôt qu'un mot de passe de connexion, mais là où une authentification par mot de passe est en jeu, elle est capturée aussi. Le flux TCP non authentifié après l'établissement est également détournable, correspondant à la technique de prise de contrôle de session Telnet/rlogin.

```bash
# capture rsh traffic from an on-path position
tcpdump -i eth0 -A port 514 -w rsh.pcap
# follow the TCP stream to read the command, output, and any data/credentials
tshark -r rsh.pcap -q -z follow,tcp,ascii,0
```

## Notes d'exploitation

- La principale valeur de capture est la commande et sa sortie, qui portent fréquemment des données sensibles ou des identifiants secondaires ; le rsh en confiance d'hôte n'a souvent aucun mot de passe à sniffer, donc le contenu est la cible.
- Le flux est détournable (injecter une commande dans la session TCP établie), la même technique que le [détournement de session rlogin](../rlogin/session-hijacking.md) et Telnet ; une seule commande injectée peut planter une confiance `.rhosts` durable.
- Nécessite une position on-path ou tap ; les r-commands sur un segment hérité plat sont facilement interceptées.
- C'est la faiblesse en clair partagée de toutes les r-commands ; voir le groupe [interception de trafic](../traffic-interception/index.md) pour le traitement transversal.

## Références

- [HackTricks: rsh](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin, same cleartext model)](https://datatracker.ietf.org/doc/html/rfc1282)
