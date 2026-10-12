---
title: "Kerberos : roasting, forge et réutilisation de tickets"
order: 3
description: "Abus de Kerberos dans Active Directory : roasting des comptes de service et des comptes sans pré-authentification pour obtenir du matériel cassable, réutilisation de clés et de tickets, forge pure et simple de tickets, abus de délégation, et les chemins vers l'impersonation que sont les shadow credentials et le spoofing de sAMAccountName."
keywords:
  - kerberos
  - kerberoasting
  - golden ticket
  - delegation
  - pass the ticket
---

# Kerberos

Kerberos est le principal protocole d'authentification d'Active Directory, et presque toute attaque AD à forte valeur finit par s'exprimer en termes Kerberos. Sa conception offre à un attaquant plusieurs ouvertures distinctes : les parties chiffrées des tickets sont dérivées de la clé d'un compte, elles peuvent donc être **roastées** et cassées hors ligne ; la clé elle-même authentifie, elle peut donc être **réutilisée** sans le mot de passe ; et quiconque détient la clé `krbtgt` ou une clé de service peut **forger** des tickets à volonté.

## Comment naissent les ouvertures

Un modèle rapide du protocole explique chaque attaque :

- Le client prouve son identité au KDC (échange AS) et reçoit un **TGT** chiffré avec la clé `krbtgt`.
- Il présente le TGT pour obtenir des **tickets de service** (échange TGS), chacun chiffré avec la clé du compte de service cible.
- Le service fait confiance au ticket parce qu'il peut le déchiffrer avec sa propre clé, et fait confiance aux privilèges qu'il contient (le PAC) parce que le KDC les a signés.

Chaque confiance de cette chaîne est une cible : la clé de compte (roasting, pass-the-key), le TGT (pass-the-ticket, golden ticket), la clé de service (silver ticket), le PAC (tickets diamond/sapphire) et les flags de délégation qui permettent à un service de demander des tickets *en tant qu'*un autre utilisateur.

## Pages

- **[Découverte de SPN](spn-discovery.md)** : trouver les comptes de service que le Kerberoasting cible.
- **[Roasting](roasting.md)** : Kerberoasting, AS-REP roasting et timeroasting, obtenir du matériel cassable depuis le protocole.
- **[Pass-the-key et overpass-the-hash](pass-the-key-and-overpass-the-hash.md)** : transformer une clé ou un hash en TGT.
- **[Pass-the-ticket](pass-the-ticket.md)** : extraire et injecter des tickets pour les réutiliser.
- **[Tickets forgés](forged-tickets.md)** : forge de tickets golden, silver, diamond et sapphire.
- **[MS14-068](ms14-068.md)** : forger un PAC privilégié, n'importe quel utilisateur vers Domain Admin sur un DC non corrigé.
- **[Délégation](delegation/index.md)** : abus de la délégation non contrainte, contrainte et contrainte basée sur les ressources.
- **[Kerberos relay](kerberos-relay.md)** : capture de TGT via la délégation non contrainte et relais de Kerberos vers LDAP ou AD CS.
- **[Shadow credentials](shadow-credentials.md)** : ajout de key credentials pour usurper une identité via PKINIT.
- **[UnPAC-the-hash](unpac-the-hash.md)** : récupérer le hash NT depuis une authentification PKINIT.
- **[Spoofing de sAMAccountName](samaccountname-spoofing.md)** : le chemin d'élévation de privilèges noPac.

## Références

- [GhostPack Rubeus: raw Kerberos abuse toolkit](https://github.com/GhostPack/Rubeus)
- [Impacket: Kerberos example scripts](https://github.com/fortra/impacket)
- [Microsoft: Kerberos authentication](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)
