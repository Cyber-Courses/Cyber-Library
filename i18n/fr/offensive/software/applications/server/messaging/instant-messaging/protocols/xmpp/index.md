---
title: "XMPP : attaquer les serveurs Jabber et leurs flux XML"
order: 2
description: "XMPP sur TCP 5222 (client, STARTTLS), 5223 (TLS historique), et 5269 (fédération serveur à serveur), servi par ejabberd, Prosody ou Openfire. La surface comprend la découverte de services non authentifiée via un flux XML ouvert, l'abus de SASL et de l'inscription in-band pour l'accès aux comptes, la rétrogradation de mécanisme vers le clair, et les failles serveur telles que le contournement d'authentification par path-traversal de la console admin d'Openfire."
keywords:
  - XMPP
  - Jabber
  - port 5222
  - ejabberd
  - Openfire
---

# XMPP

XMPP (Jabber) est un protocole de messagerie à flux XML. Les clients se connectent sur TCP 5222 avec STARTTLS, les clients historiques sur 5223 avec TLS implicite, et les serveurs se fédèrent entre eux sur 5269. Les implémentations serveur courantes sont ejabberd, Prosody et Openfire ; le protocole est identique entre elles, mais les failles serveur ne le sont pas, donc identifier le démon est la première étape. Le protocole est du XML conversationnel : vous envoyez un `<stream:stream>` d'ouverture et échangez des stanzas `<iq>`, `<message>` et `<presence>`. Une grande partie de la valeur offensive ne nécessite aucun compte, car la découverte de services, l'annonce de fonctionnalités et l'inscription in-band répondent toutes sur un flux non authentifié.

Ouvrez un flux pour identifier le serveur :

```bash
nmap -p5222,5269 -sV --script xmpp-info <target>

# open a client stream by hand (the server echoes its features and often its software)
printf "<?xml version='1.0'?><stream:stream to='target.lan' xmlns='jabber:client' xmlns:stream='http://etherx.jabber.org/streams' version='1.0'>" | nc <target> 5222
```

Le `<stream:features>` renvoyé liste les mécanismes SASL et si STARTTLS est exigé ou seulement proposé ; le `<stream:stream from='target.lan' ...>` et toute bannière logicielle distinguent ejabberd, Prosody et Openfire.

## Triage

- Le flux s'ouvre-t-il et annonce-t-il des fonctionnalités sans que TLS ne soit imposé ? Cela permet la rétrogradation de mécanisme et la capture en clair, couvertes dans l'[authentification](authentication.md).
- La découverte de services répond-elle sur un flux non authentifié, et l'inscription in-band est-elle activée ? Cela pilote l'[énumération](enumeration.md) et l'auto-provisionnement.
- De quel démon s'agit-il, et est-ce une version d'Openfire présentant le contournement d'authentification par path-traversal de la console admin ? Cela pilote l'[exploitation serveur](server-exploitation.md).

## Sous-thèmes

- **[Énumération](enumeration.md)** : ouvrez un flux XML et utilisez disco#info/disco#items pour lister les fonctionnalités, composants et vhosts du serveur, et sondez l'existence des utilisateurs via les vérifications de liste de contacts, de vCard et d'inscription.
- **[Authentification](authentication.md)** : SASL (PLAIN/SCRAM/DIGEST-MD5), inscription in-band pour auto-provisionner des comptes, rétrogradation de mécanisme vers le clair, et brute force SASL.
- **[Exploitation serveur](server-exploitation.md)** : le contournement d'authentification par path-traversal de la console admin d'Openfire chaîné à l'upload de plugin malveillant pour la RCE, ainsi que les problèmes de modules et de TLS d'ejabberd et Prosody.

## Références

- [RFC 6120 (XMPP Core)](https://datatracker.ietf.org/doc/html/rfc6120)
- [RFC 6121 (XMPP Instant Messaging and Presence)](https://datatracker.ietf.org/doc/html/rfc6121)
- [XEP-0030 Service Discovery](https://xmpp.org/extensions/xep-0030.html)
