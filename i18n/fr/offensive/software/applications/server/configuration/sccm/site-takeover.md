---
title: "Prise de contrôle d'un site SCCM : relayage vers Full Administrator"
order: 3
description: "Prendre le contrôle d'un site Configuration Manager en forçant l'authentification du serveur de site et en la relayant vers la base de données du site (MSSQL) ou vers le SMS Provider, ce qui octroie le rôle Full Administrator et, avec lui, l'exécution en tant que SYSTEM sur chaque appareil géré."
keywords:
  - prise de contrôle de site SCCM
  - relayage NTLM
  - base de données de site MSSQL
  - SMS Provider
  - Full Administrator
---

# Prise de contrôle d'un site

Obtenir le rôle **Full Administrator** sur un site est le gain ultime : c'est l'exécution de code à distance en tant que SYSTEM sur **chaque appareil du site**. La voie moderne ne nécessite aucune identifiant SCCM. Elle force le compte machine du **serveur de site** à s'authentifier et [relaie](../../directory/active-directory/authentication/ntlm/relay.md) cette authentification vers un composant qui lui fait confiance, soit la base de données du site, soit le SMS Provider.

## Relayage vers la base de données du site (MSSQL)

Le compte machine du serveur de site est sysadmin sur sa propre **base de données de site**. Forcez-le à s'authentifier, relayez vers MSSQL et écrivez-vous un enregistrement Full Administrator :

```bash
# Relay the coerced site-server auth to the site database and add an SCCM admin
SharpSCCM.exe ... # enumerate the site database target first
ntlmrelayx.py -t mssql://<site-db> -q "<SCCM admin insert>" -smb2support
# coerce the site server (PetitPotam / Coercer) so its machine account authenticates to the relay
```

## Relayage vers le SMS Provider

Le **SMS Provider** est la couche WMI/administration ; relayer une authentification privilégiée vers lui, via SMB ou HTTP, permet de même d'enregistrer un nouveau Full Administrator, et c'est la voie à suivre lorsque la base de données n'est pas directement accessible.

## Prise de contrôle de la hiérarchie

Lorsqu'un **central administration site** se trouve au-dessus de sites primaires, prendre le sommet de la hiérarchie fait redescendre le rôle Full Administrator vers chaque site primaire enfant, alors notez la hiérarchie lors de la [reconnaissance](reconnaissance.md) et visez le site le plus élevé accessible.

## Notes d'exploitation

- La coercition cible le **compte machine du serveur de site**, donc cela ne nécessite aucun rôle SCCM et souvent rien de plus qu'un accès réseau ainsi qu'une primitive de coercition.
- Cela s'enchaîne directement avec la chaîne d'outils de relayage AD : les mêmes [coercition](../../directory/active-directory/authentication/ntlm/coercion.md) et `ntlmrelayx` que vous utilisez contre un DC s'appliquent ici, seule la cible du relayage change.
- Full Administrator équivaut à **SYSTEM sur chaque client** via le [déploiement d'applications](application-deployment.md), donc la prise de contrôle d'un site est de fait une compromission à l'échelle de tout le parc, et pas seulement un contrôle de SCCM.
- Lorsque le relayage est bloqué, repliez-vous sur les voies de [récupération d'identifiants](credential-harvesting.md) ou relayez plutôt le serveur de site vers [AD CS](../../directory/active-directory/authentication/certificates/index.md).

## Outils

- **SharpSCCM / sccmhunter** : identifier la base de données du site et le SMS Provider, et ajouter des administrateurs après le relayage.
- **ntlmrelayx.py** : relayer l'authentification forcée vers MSSQL ou le SMS Provider.
- **Coercer / PetitPotam** : forcer le compte machine du serveur de site à s'authentifier.

## Références

- [SpecterOps : Misconfiguration Manager, TAKEOVER](https://github.com/subat0mik/Misconfiguration-Manager/tree/main/attack-techniques/TAKEOVER)
- [Truesec : SCCM tier killer](https://www.truesec.com/hub/blog/sccm-tier-killer)
- [logan-goins : attaquer et défendre Configuration Manager](http://logan-goins.com/2025-04-25-sccm/)
