---
title: "Microsoft Exchange : attaquer le serveur de messagerie on-premises"
order: 3
description: "La surface d'attaque de Microsoft Exchange on-premises, organisée du périmètre vers Active Directory : lire la version du serveur depuis OWA et ECP pour choisir une chaîne, énumération et spraying d'utilisateurs, les chaînes de RCE front-end vers back-end, la coercition PrivExchange vers AD, l'accès aux boîtes aux lettres, l'abus de client et la persistance dans le flux de messagerie."
keywords:
  - Exchange
  - OWA
  - ECP
  - ProxyShell
  - PrivExchange
  - X-OWA-Version
---

# Microsoft Exchange

Microsoft Exchange on-premises se situe à la jonction entre le périmètre et Active Directory. Les services Client Access (front-end) sont exposés sur internet en HTTPS (`/owa`, `/ecp`, `/ews`, `/Microsoft-Server-ActiveSync`, `/autodiscover`, `/mapi`, `/rpc`, `/powershell`), chacun d'entre eux s'authentifie contre le domaine, et les serveurs Exchange ainsi que leurs groupes de sécurité détiennent **de hauts privilèges dans AD par conception**, historiquement un accès en écriture sur l'objet domaine lui-même. C'est cette combinaison qui fait d'Exchange l'une des cibles les plus productives dans un parc Windows : le front-end atteint un back-end privilégié, un point d'ancrage sur la machine s'exécute en tant que `SYSTEM`, et même sans exécution de code le serveur peut être contraint de s'authentifier auprès de vous. Les boîtes aux lettres sont à elles seules un trésor.

Tout sur cette surface dépend d'un fait que vous établissez en premier : **le numéro de build exact**. Les chaînes de RCE pré-authentification et à identifiant unique sont étroitement liées à la version, et déclencher la mauvaise contre une build corrigée ne fait que brûler la mission.

## Triage : lire la build, puis choisir la chaîne

Le numéro de build fuite sans authentification à plusieurs endroits. OWA sert ses ressources statiques depuis un chemin versionné, et le serveur front-end estampille sa version dans les en-têtes de réponse :

```bash
# Header stamp (X-OWA-Version on newer CUs, X-FEServer names the front-end host)
curl -sk -I 'https://mail.example.com/owa/auth/logon.aspx' | grep -iE 'X-OWA-Version|X-FEServer'

# Versioned static path in the logon HTML: /owa/auth/15.2.1544.4/... IS the build
curl -sk 'https://mail.example.com/owa/auth/logon.aspx' | grep -oE '/owa/auth/[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/' | head -1

# ECP login page carries the same versioned path and confirms /ecp is reachable
curl -sk 'https://mail.example.com/ecp/' | grep -oE '/ecp/[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/' | head -1
```

Les premiers numéros nomment la famille de produit, et la build complète décide quelle chaîne est à portée :

- **15.0.x** = Exchange 2013, **15.1.x** = Exchange 2016, **15.2.x** = Exchange 2019. Un `15.0`/`15.1`/`15.2` nu sans la build complète en notation pointée signifie généralement une CU très ancienne qui n'a jamais nettoyé le chemin.
- Une build **au niveau ou en dessous du rollup de sécurité du début 2021** est à portée de **[ProxyLogon](rce-chains/proxylogon.md)** (SSRF pré-authentification vers web shell, `SYSTEM`, sans identifiant).
- Une build **en dessous du rollup de mi-2021** est à portée de **[ProxyShell](rce-chains/proxyshell.md)** (confusion de chemin Autodiscover pré-authentification vers le back-end PowerShell, sans identifiant).
- Une build **en dessous du rollup de fin 2022** est à portée de **[ProxyNotShell](rce-chains/proxynotshell.md)** (nécessite un identifiant de boîte aux lettres valide) ; si la mitigation de réécriture d'URL a été appliquée mais que la machine n'a jamais été entièrement corrigée, la variante **OWASSRF** de la même page fonctionne encore.
- Un `/ecp` atteignable avec une `machineKey` connue ou fuitée est à portée de la **[désérialisation ViewState](rce-chains/viewstate-deserialization.md)** quelle que soit la famille, une fois que vous détenez une session ECP valide ou les clés statiques.

Faites d'abord correspondre la build, puis lisez les pages de chaînes ci-dessous. Lorsqu'aucune chaîne ne convient, la surface reste entièrement attaquable depuis le périmètre via l'énumération, le spraying et la coercition.

## L'arc d'attaque

1. **Énumérer** la build, les utilisateurs valides et le carnet d'adresses depuis l'extérieur.
2. **Sprayer** les endpoints web pour transformer une liste d'utilisateurs en identifiant de domaine.
3. **Exécuter** une chaîne de RCE adaptée à la build pour obtenir `SYSTEM`, ou **contraindre** le serveur avec PrivExchange et le relayer vers AD lorsqu'aucune chaîne ne convient.
4. **Lire le courrier** à grande échelle via l'usurpation d'identité, et **abuser des clients Outlook** lorsque vous détenez une boîte aux lettres mais pas le serveur.
5. **Persister** dans le flux de messagerie avec des règles de transport et de transfert qui survivent aux réinitialisations de mot de passe.

## Pages et sous-thèmes

- **[Énumération](enumeration.md)** : découverte de domaine et d'utilisateurs via Autodiscover, validation de nom d'utilisateur basée sur le timing d'OWA/EWS, extraction du GAL/OAB, fuite de version via ECP, et fuite de nom interne via le type-2 NTLM depuis `/rpc` et `/ews`.
- **[Password spraying](password-spraying.md)** : spraying d'OWA, EWS, Autodiscover et ActiveSync, cadence tenant compte du verrouillage, et les endpoints qui ignorent souvent le suivi du verrouillage.
- **[Chaînes de RCE](rce-chains/index.md)** : le motif proxy front-end vers back-end privilégié, avec ProxyLogon, ProxyShell, ProxyNotShell/OWASSRF et la désérialisation ViewState.
- **[PrivExchange](privexchange.md)** : contraindre le compte machine Exchange à s'authentifier et le relayer vers LDAP pour un DCSync.
- **[Accès aux boîtes aux lettres](mailbox-access.md)** : usurpation EWS, export de boîtes aux lettres via Exchange Management Shell, et recherche de courrier à l'échelle de l'organisation après un identifiant ou le serveur.
- **[Abus de client Outlook](outlook-client-abuse.md)** : règles, formulaires et pages d'accueil de dossier qui exécutent du code sur le poste de travail de la victime via MAPI/EWS, sans RCE serveur nécessaire.
- **[Persistance dans le flux de messagerie](mail-flow-persistence.md)** : règles de transport, journalisation et transfert qui copient silencieusement le courrier de l'organisation vers une adresse externe.

## Références

- [MailSniper (dafthack): Exchange enumeration, spraying, and mailbox search](https://github.com/dafthack/MailSniper)
- [Microsoft: Exchange Server build numbers and release dates](https://learn.microsoft.com/en-us/exchange/new-features/build-numbers-and-release-dates)
- [Black Hills InfoSec: attacking Exchange with MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
- [Google Cloud / Mandiant: ProxyShell exploiting Microsoft Exchange servers](https://cloud.google.com/blog/topics/threat-intelligence/pst-want-shell-proxyshell-exploiting-microsoft-exchange-servers)
