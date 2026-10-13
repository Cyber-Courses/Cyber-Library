---
title: "Protocoles : attaquer les protocoles de chat ouverts en temps réel et leurs serveurs"
description: "Protocoles de chat ouverts en temps réel avec de multiples implémentations serveur indépendantes : IRC sur 6667/6697, XMPP sur 5222/5223/5269, et Matrix sur 8008/8448/443. Chacun expose une surface en libre-service (énumération, provisionnement de comptes, abus d'authentification) et des failles serveur spécifiques à l'implémentation. Couvre comment identifier quel protocole et quel démon s'exécutent, et où aller ensuite."
keywords:
  - instant messaging protocols
  - IRC
  - XMPP
  - Matrix
  - chat server
---

# Protocoles

Ce sont les protocoles de chat en temps réel ouverts et standardisés, chacun avec plusieurs implémentations serveur indépendantes plutôt qu'un produit d'un seul éditeur. Cette forme oriente l'approche offensive : le protocole est public, donc vous le parlez directement sur une socket, et la valeur réside dans la surface en libre-service (inscription ouverte, énumération de la liste de contacts et de l'annuaire, rétrogradation de mécanisme) ainsi que dans les bogues propres au démon déployé. Les trois protocoles fédèrent ou relient des serveurs, donc un seul nœud compromis atteint souvent un réseau plus large d'utilisateurs et de salons.

## Triage

Un seul balayage vous dit quel protocole, et la bannière ou le handshake de flux vous dit quel démon :

```bash
nmap -sV -p6667,6697,5222,5269,8448 <target>
#  6667 IRC plaintext   6697 IRC over TLS
#  5222 XMPP client (STARTTLS)   5223 XMPP legacy TLS   5269 XMPP server-to-server
#  8008 Matrix client-server     8448 Matrix federation     443 Matrix behind a reverse proxy
```

Identifiez en parlant le protocole :

- IRC répond à un handshake brut `NICK`/`USER` avec des réponses numériques et une réponse `VERSION` nommant l'ircd (UnrealIRCd, InspIRCd, Charybdis, ngIRCd). Voir [IRC](irc/index.md).
- XMPP répond à un `<stream:stream>` d'ouverture avec un bloc `<stream:features>` dont le `from` et les espaces de noms nomment le serveur (ejabberd, Prosody, Openfire). Voir [XMPP](xmpp/index.md).
- Matrix répond à `GET /_matrix/client/versions` et `/_matrix/federation/v1/version` avec du JSON nommant Synapse, Dendrite ou Conduit. Voir [Matrix](matrix/index.md).

Pour chacun, les questions récurrentes sont les mêmes : pouvez-vous énumérer les utilisateurs, les salons et les fonctionnalités du serveur sans identifiants ; pouvez-vous vous auto-provisionner ou vous authentifier en tant que quelqu'un ; et le démon spécifique présente-t-il une faille serveur connue.

## Sous-thèmes

- **[IRC](irc/index.md)** : protocole texte sur 6667/6697, salons, services NickServ/ChanServ, et opérateurs IRC, avec des versions d'ircd piégées et vulnérables.
- **[XMPP](xmpp/index.md)** : flux XML sur 5222/5223/5269, découverte de services, SASL et inscription in-band, et failles serveur dans ejabberd, Prosody et Openfire.
- **[Matrix](matrix/index.md)** : API HTTP/JSON sur 8008/8448/443, inscription ouverte et administration par secret partagé, SSRF de média, et abus de fédération contre Synapse, Dendrite et Conduit.

## Références

- [RFC 1459 (Internet Relay Chat Protocol)](https://datatracker.ietf.org/doc/html/rfc1459)
- [RFC 6120 (XMPP Core)](https://datatracker.ietf.org/doc/html/rfc6120)
- [Matrix specification](https://spec.matrix.org/latest/)
- [HackTricks: pentesting network services](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
