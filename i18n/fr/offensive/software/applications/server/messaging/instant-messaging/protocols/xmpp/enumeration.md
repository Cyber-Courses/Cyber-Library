---
title: "Énumération : découverte de services et sondage des utilisateurs via un flux XML"
description: "Un flux XMPP ouvert répond à la découverte de services (disco#info et disco#items) sans identifiants, listant les fonctionnalités prises en charge par le serveur et ses composants tels que le chat multi-utilisateurs, les proxies et les interfaces d'administration. Les vhosts hébergés, et l'existence d'utilisateurs spécifiques, sont sondés via des requêtes de composants, des requêtes vCard et de liste de contacts, et des vérifications d'inscription in-band."
keywords:
  - XMPP enumeration
  - service discovery
  - disco#info
  - disco#items
  - XMPP components
---

# Énumération

La découverte de services XMPP (XEP-0030) est conçue pour permettre aux clients d'apprendre ce qu'un serveur sait faire, et elle répond sur un flux qui ne s'est pas authentifié. Deux requêtes la portent : `disco#info` renvoie les fonctionnalités prises en charge par un serveur ou un composant (identités et espaces de noms de fonctionnalités), et `disco#items` renvoie les éléments enfants, qui sur un serveur sont ses composants, le service de chat multi-utilisateurs hébergé, les proxies de transfert de fichiers, et souvent un composant admin. À partir de là, vous apprenez le logiciel serveur, les vhosts hébergés, et si des capacités comme l'inscription in-band sont activées, le tout avant même d'avoir un compte. L'existence des utilisateurs est ensuite sondée avec des stanzas ciblées.

## Prérequis

Un flux ouvert (le serveur a répondu à votre `<stream:stream>` avec `<stream:features>`). La découverte fonctionne avant TLS et avant SASL sur la plupart des déploiements ; si le serveur renvoie `<not-authorized/>` à la découverte, il requiert une session authentifiée, auquel cas auto-provisionnez-en une d'abord via l'[authentification](authentication.md). Connaissez le domaine du serveur (le `to=` du flux), car les composants sont nommés comme des sous-domaines de celui-ci.

## Découverte de services détaillée

Envoyez un `<iq>` de type `get` portant la requête disco#items, puis creusez dans chaque composant renvoyé avec disco#info :

```xml
<!-- list the server's components and services -->
<iq type='get' id='d1' to='target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#items'/>
</iq>

<!-- server response: each item is a component you can target next -->
<iq type='result' id='d1' from='target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#items'>
    <item jid='conference.target.lan' name='Chatrooms'/>
    <item jid='proxy.target.lan' name='SOCKS5 Bytestreams'/>
    <item jid='pubsub.target.lan' name='Publish-Subscribe'/>
  </query>
</iq>

<!-- ask a component what it supports and what it is -->
<iq type='get' id='d2' to='target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#info'/>
</iq>
<iq type='result' id='d2' from='target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#info'>
    <identity category='server' type='im' name='Prosody'/>
    <feature var='jabber:iq:register'/>
    <feature var='http://jabber.org/protocol/muc'/>
  </query>
</iq>
```

Interprétation : chaque `<item>` est un composant actif, nommant le service de chat multi-utilisateurs (`conference.`), le proxy de fichiers et le nœud pubsub à énumérer plus avant. Le `<identity ... name='Prosody'/>` identifie le démon pour l'[exploitation serveur](server-exploitation.md), et la `<feature var='jabber:iq:register'/>` annoncée confirme que l'inscription in-band est activée, de sorte que vous pouvez vous auto-provisionner un compte sans aucun identifiant (voir [authentification](authentication.md)).

Énumérez les salons de chat sur le composant MUC de la même manière :

```xml
<iq type='get' id='r1' to='conference.target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#items'/>
</iq>
<!-- each returned <item jid='ops@conference.target.lan' name='ops'/> is a room to join -->
```

## Sonder l'existence des utilisateurs et des vhosts

```xml
<!-- does this user exist? a vCard request distinguishes a real JID from an empty one -->
<iq type='get' id='v1' to='jdoe@target.lan'><vCard xmlns='vcard-temp'/></iq>
<!-- type='result' with content => user exists; item-not-found => no such account -->

<!-- is in-band registration open, and what fields does it want? -->
<iq type='get' id='reg1' to='target.lan'><query xmlns='jabber:iq:register'/></iq>
<!-- a returned <username/><password/> form => self-provisioning available -->
```

Une vCard `type='result'` avec des champs remplis (ou des données de présence/dernière activité) confirme que le JID est réel, tandis que `<item-not-found/>` ou un résultat vide signifie qu'il ne l'est pas ; itérez une liste de noms pour construire l'inventaire des utilisateurs. Les vhosts supplémentaires apparaissent comme des domaines serveur additionnels dans la découverte ou dans les fonctionnalités du flux ; répétez la découverte contre chacun.

## Suite

- La liste des composants nomme exactement quoi attaquer ensuite : le service MUC pour les salons, le composant admin pour les opérations privilégiées, et tout composant HTTP-upload ou proxy pour l'abus de fichiers.
- Un formulaire `jabber:iq:register` confirmé signifie que vous pouvez créer votre propre compte immédiatement ; allez à l'[authentification](authentication.md).
- Les JID énumérés alimentent le brute force SASL et la liste des vrais utilisateurs à usurper une fois que vous détenez un compte.

## Références

- [XEP-0030 Service Discovery](https://xmpp.org/extensions/xep-0030.html)
- [XEP-0077 In-Band Registration](https://xmpp.org/extensions/xep-0077.html)
- [RFC 6121 (XMPP IM and Presence)](https://datatracker.ietf.org/doc/html/rfc6121)
