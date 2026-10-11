---
title: "SMTP : attaquer le service de transfert de courrier"
order: 1
description: "SMTP sur les ports 25, 587 et 465 est la surface de transport du courrier : il révèle les destinataires valides, relaie le courrier de n'importe qui lorsqu'il est mal configuré, authentifie les clients de soumission en clair, peut être rétrogradé hors de TLS, peut être smuggled pour usurper des expéditeurs de confiance, et exécute un logiciel MTA (Exim, Sendmail, Postfix) avec son propre historique d'exploitation."
keywords:
  - smtp
  - port 25
  - mail transfer agent
  - open relay
  - smtp enumeration
---

# SMTP

SMTP (Simple Mail Transfer Protocol) déplace le courrier entre serveurs et des clients vers les serveurs. Il écoute sur trois ports aux rôles différents : TCP 25 pour le transfert de serveur à serveur (MTA à MTA), TCP 587 pour la soumission authentifiée de clients, et TCP 465 pour la soumission sur TLS implicite. Le protocole est un dialogue de verbes en texte clair ligne par ligne, de sorte que tout ce qu'il fait est directement observable et directement exploitable : les verbes qui valident les destinataires révèlent la liste des utilisateurs, un serveur qui transfère du courrier qu'il devrait refuser est un open relay, les ports de soumission transportent des identifiants potentiellement interceptables, le STARTTLS opportuniste peut être dépouillé, et le traitement de la fin de données peut être smuggled à travers un relais confiant. Sous le protocole se trouve une implémentation MTA dont la bannière le nomme et dont la version décide quel exploit côté serveur s'applique.

## Identifier et trier

Un scan plus un EHLO piloté à la main vous indiquent le MTA, les verbes annoncés et quelle page enfant s'applique :

```bash
nmap -p25,465,587 -sV --script smtp-commands,smtp-open-relay,smtp-ntlm-info <target>
# manual: read the banner and the advertised capabilities
nc <target> 25
# 220 mail.victim.com ESMTP Postfix            <- banner names the MTA
EHLO x
# 250-mail.victim.com
# 250-SIZE 52428800
# 250-VRFY                                     <- user-enumeration verb offered
# 250-STARTTLS                                 <- opportunistic TLS, downgrade candidate
# 250-AUTH LOGIN PLAIN                          <- credential surface on this port
# 250 8BITMIME
```

Lisez le résultat et routez : la chaîne de la bannière (Postfix / Exim / Sendmail / Microsoft ESMTP) pointe vers l'[exploitation du serveur](server-exploitation/index.md) ; `VRFY`/`EXPN` ou toute différence de réponse par destinataire pointe vers l'[énumération des utilisateurs](user-enumeration.md) ; `AUTH` sur 25/587 pointe vers l'[authentification](authentication.md) ; `STARTTLS` annoncé sur 25 pointe vers le [downgrade STARTTLS](starttls-downgrade.md) ; une disposition à accepter un MAIL FROM et un RCPT TO externes pointe vers l'[open relay](open-relay.md). Le SMTP smuggling se teste contre le serveur entrant quel que soit ce qu'annonce EHLO.

## Sous-rubriques

- **[Énumération des utilisateurs](user-enumeration.md)** : valider les destinataires avec VRFY, EXPN et le sondage RCPT TO.
- **[Open relay](open-relay.md)** : envoyer du courrier au nom de n'importe qui à travers un serveur qui relaie pour des expéditeurs externes.
- **[Authentification](authentication.md)** : pulvérisation d'identifiants AUTH LOGIN/PLAIN et capture en clair sur la soumission.
- **[Downgrade STARTTLS](starttls-downgrade.md)** : dépouiller le TLS opportuniste et injecter des commandes avant le handshake.
- **[SMTP smuggling](smtp-smuggling.md)** : exploiter le désaccord sur la fin de données pour injecter des transactions usurpées passant DMARC.
- **[Exploitation du serveur](server-exploitation/index.md)** : attaques spécifiques aux MTA contre Exim, Sendmail et Postfix.

## Références

- [RFC 5321 (Simple Mail Transfer Protocol)](https://datatracker.ietf.org/doc/html/rfc5321)
- [HackTricks : SMTP (25,465,587)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
- [Nmap NSE : scripts smtp](https://nmap.org/nsedoc/categories/default.html)
