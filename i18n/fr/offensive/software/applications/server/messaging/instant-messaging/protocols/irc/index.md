---
title: "IRC : attaquer les serveurs Internet Relay Chat"
order: 1
description: "IRC sur TCP 6667 en clair et 6697 sur TLS, servi par des implémentations d'ircd telles qu'UnrealIRCd, InspIRCd et Charybdis. La surface comprend l'énumération non authentifiée via des sessions enregistrées, les services NickServ/ChanServ et le privilège d'opérateur OPER, ainsi que les failles serveur dont la distribution piégée d'UnrealIRCd 3.2.8.1."
keywords:
  - IRC
  - ircd
  - port 6667
  - IRC operator
  - UnrealIRCd
---

# IRC

Internet Relay Chat est un protocole texte orienté ligne sur TCP 6667 (en clair) et 6697 (TLS). Un serveur est un ircd (UnrealIRCd, InspIRCd, Charybdis, ngIRCd, ircd-hybrid) qui héberge des salons, se lie à d'autres serveurs pour former un réseau, et exécute des services compagnons (NickServ, ChanServ) pour la propriété des comptes et des salons. Deux niveaux de privilège comptent du point de vue offensif : un utilisateur enregistré normal qui peut énumérer tout le réseau, et un opérateur IRC (oper) qui détient des commandes serveur allant jusqu'au chargement de modules et à l'exécution sur l'hôte. Les salons eux-mêmes véhiculent fréquemment des identifiants, des interfaces de commande de bots et des échanges internes.

Ouvrir une session est trivial, et les premières lignes identifient déjà le démon :

```bash
nmap -p6667,6697 -sV --script irc-info,irc-unrealircd-backdoor <target>

nc <target> 6667
NICK pentest
USER pentest 0 * :pentest
# the server replies with numerics: 001 (welcome), 002/003/004 (server + ircd version),
# 005 (ISUPPORT feature list). 004 names the daemon, e.g. "UnrealIRCd-6" or "InspIRCd-3".
```

Un `464 :Password required` en réponse signifie que le serveur a un mot de passe de connexion (`PASS`), et un `ERROR :Closing link` immédiat avec un message de K-line signifie que votre adresse source est bannie.

## Triage

- Le handshake se termine-t-il sans `PASS`, et que disent les numériques `004`/`005` sur l'ircd et sa version ? Cela décide si une faille serveur connue s'applique.
- Pouvez-vous lister (`LIST`) les salons et faire `WHO *`/`WHOIS` sur les utilisateurs sans être opérateur ? Cela pilote l'[énumération](enumeration.md).
- NickServ et ChanServ sont-ils présents, et pouvez-vous atteindre le statut d'opérateur via `OPER` ? Cela pilote l'[abus des services et des opérateurs](services-and-operator-abuse.md).
- Le démon est-il une version avec une porte dérobée ou une faille de module connue ? Cela pilote l'[exploitation serveur](server-exploitation.md).

## Sous-thèmes

- **[Énumération](enumeration.md)** : enregistrez une session, puis lisez le réseau avec `VERSION`, `LUSERS`, `LIST`, `WHO`, `WHOIS`, `NAMES`, `STATS` et `MAP`, ainsi que les requêtes de services.
- **[Abus des services et des opérateurs](services-and-operator-abuse.md)** : prise de contrôle de NickServ/ChanServ, SASL, et escalade vers opérateur IRC via `OPER` et des O:lines faibles.
- **[Exploitation serveur](server-exploitation.md)** : la distribution piégée d'UnrealIRCd 3.2.8.1, les problèmes de modules et de TLS d'InspIRCd et Charybdis, et l'abus de DCC.

## Références

- [RFC 1459 (Internet Relay Chat Protocol)](https://datatracker.ietf.org/doc/html/rfc1459)
- [RFC 2812 (IRC Client Protocol)](https://datatracker.ietf.org/doc/html/rfc2812)
- [HackTricks: pentesting IRC (6667)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-irc.html)
