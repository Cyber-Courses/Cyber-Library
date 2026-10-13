---
order: 1
title: "Identifiants : extraire, casser et deviner les secrets Active Directory"
description: "Obtenir le matériel d'identification dans Active Directory : extraire les secrets de LSASS, de la SAM, de DPAPI et de la base NTDS (y compris DCSync), casser hors ligne les hash récupérés, et deviner les mots de passe par spraying."
keywords:
  - credential dumping
  - LSASS
  - NTDS
  - DCSync
  - password spraying
---

# Identifiants

Les identifiants sont la monnaie d'une attaque Active Directory. Cette section couvre les trois façons de les acquérir : **extraire** les secrets déjà présents sur un hôte ou dans l'annuaire, **casser** hors ligne les hash récupérés en mots de passe utilisables, et les **deviner** contre le domaine par spraying. Tout le reste de la section authentification et identifiants consomme ces secrets ou en produit davantage.

## Où résident les secrets

Un hôte Windows compromis et l'annuaire lui-même détiennent plusieurs magasins d'identifiants :

- **La mémoire de LSASS** : le texte clair, les hash NTLM et les tickets Kerberos de chaque principal ayant une session sur l'hôte.
- **La SAM et les secrets LSA** : les hash de comptes locaux et les secrets de comptes de service et machine mis en cache dans le registre.
- **Les magasins protégés par DPAPI** : mots de passe de navigateur et d'applications enregistrés, et identifiants dans le coffre Windows.
- **La base NTDS** : sur un contrôleur de domaine (ou via DCSync depuis n'importe quel hôte disposant du droit de réplication), les hash NTLM de chaque compte du domaine, y compris `krbtgt`.
- **L'annuaire** : mots de passe dans les attributs, mots de passe gérés LAPS et gMSA (couverts sous [identifiants dans les attributs](../../../ldap/protocol/credentials-in-attributes.md)).

## Pages

- **[Usurpation de jeton](token-impersonation.md)** : SeImpersonate vers SYSTEM depuis un point d'ancrage de compte de service (la famille potato).
- **[Extraction de LSASS](lsass-dumping.md)** : extraire les identifiants et tickets vivants de la mémoire de LSASS.
- **[Secrets SAM et LSA](sam-and-lsa-secrets.md)** : hash locaux et secrets machine et de service mis en cache.
- **[Secrets DPAPI](dpapi-secrets.md)** : identifiants de navigateur, de coffre-fort et d'applications.
- **[NTDS et DCSync](ntds-and-dcsync.md)** : les hash de tout le domaine depuis le DC ou par réplication.
- **[ZeroLogon](zerologon.md)** : réinitialiser le compte machine du DC via Netlogon pour une prise de contrôle du domaine non authentifiée.
- **[RODC](rodc.md)** : extraction d'identifiants mis en cache et le golden ticket restreint depuis un contrôleur de domaine en lecture seule.
- **[Cracking](cracking.md)** : transformer hors ligne les hash récupérés en mots de passe.
- **[Découverte des hôtes et du domaine](host-and-domain-discovery.md)** : trouver les contrôleurs de domaine, les contextes de nommage et la configuration du domaine.
- **[Énumération LDAP](ldap-enumeration.md)** : interroger l'annuaire pour les objets, attributs et drapeaux exploitables.
- **[Énumération des utilisateurs et des groupes](user-and-group-enumeration.md)** : constituer la liste des comptes à cibler.
- **[Énumération des sessions](session-enumeration.md)** : localiser les utilisateurs connectés à cibler pour le vol d'identifiants et le déplacement latéral.
- **[Sessions nulles](null-sessions.md)** : énumération SMB/LDAP anonyme et RID cycling, avant tout identifiant.
- **[Politique de mot de passe](password-policy.md)** : lire la politique qui borne un spraying sûr.
- **[Password spraying](password-spraying.md)** : deviner des identifiants valides sans verrouillage.
- **[Ordinateurs pré-créés](pre-created-computers.md)** : prendre le contrôle d'objets ordinateur pré-provisionnés avec leur nom-comme-mot-de-passe par défaut.

Une fois que vous détenez les secrets de l'annuaire, le même accès soutient la **persistance de domaine** :

- **[DCShadow](dcshadow.md)** : pousser des modifications arbitraires via un contrôleur de domaine malveillant.
- **[DSRM](dsrm.md)** : le compte de porte dérobée local du contrôleur de domaine.
- **[Skeleton Key](skeleton-key.md)** : un mot de passe maître injecté dans LSASS.
- **[SSP personnalisé](custom-ssp.md)** : journaliser les identifiants en clair sur l'hôte.

## Références

- [adsecurity: Mimikatz and command reference](https://adsecurity.org/?page_id=1821)
- [Impacket (fortra): secretsdump and the credential toolkit](https://github.com/fortra/impacket)
- [NetExec (Pennyw0rth)](https://github.com/Pennyw0rth/NetExec)
- [The Hacker Recipes: Active Directory movement](https://www.thehacker.recipes/ad/movement/)
