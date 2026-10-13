---
title: "Énumération des ACL : trouver les entrées de contrôle d'accès exploitables"
order: 1
description: "Énumération des DACL des objets Active Directory pour trouver les entrées de contrôle d'accès qu'un principal contrôlé peut exploiter : GenericAll, WriteDacl, WriteOwner, GenericWrite et les droits étendus qui mènent à l'élévation de privilèges."
keywords:
  - énumération des ACL
  - DACL
  - GenericAll
  - WriteDacl
  - droits étendus
---

# Énumération des ACL

Chaque objet AD porte une liste de contrôle d'accès discrétionnaire (DACL) composée d'entrées de contrôle d'accès (ACE) qui indiquent quels principaux peuvent le lire ou le modifier. Les ACE mal configurées constituent le chemin d'élévation de privilèges le plus courant dans AD : un principal peu privilégié disposant de la bonne ACE sur un objet privilégié peut réinitialiser un mot de passe, s'ajouter à un groupe ou s'en approprier la propriété. L'énumération consiste ici à trouver, à partir de ce que vous contrôlez, sur quels objets vous détenez des droits dangereux.

## Ce qu'il faut rechercher

Les droits à forte valeur, par ordre approximatif de puissance :

- **`GenericAll`** : contrôle total (réinitialiser le mot de passe, ajouter à un groupe, écrire n'importe quel attribut).
- **`WriteDacl`** : réécrire la DACL de l'objet, puis vous accorder `GenericAll`.
- **`WriteOwner`** : s'approprier la propriété, puis réécrire la DACL.
- **`GenericWrite` / `WriteProperty`** : écrire des attributs précis, `servicePrincipalName` (roasting ciblé), `msDS-KeyCredentialLink` (shadow credentials), `msDS-AllowedToActOnBehalfOfOtherIdentity` (RBCD), `member` / `primaryGroupID` (appartenance à un groupe).
- **Droits étendus et de lecture** : `User-Force-Change-Password` (réinitialisation sans l'ancien mot de passe), `DS-Replication-Get-Changes` + `...-All` (le droit DCSync, sur l'objet domaine) et les droits de lecture sur `msDS-ManagedPassword` / `ms-Mcs-AdmPwd` qui exposent les secrets gMSA et LAPS.
- **Tous les droits étendus** : une ACE qui accorde un droit étendu avec le **GUID tout à zéro** (`00000000-0000-0000-0000-000000000000`) signifie *chaque* droit étendu à la fois, incluant silencieusement DCSync, la réinitialisation de mot de passe et la lecture LAPS ; lisez donc une telle ACE comme équivalant à l'ensemble d'entre eux combinés.
- **`WriteGPLink`** sur une OU : y lier une GPO (traité sous [Group Policy](../group-policy/index.md)).

La [vue d'ensemble des DACL](index.md) associe chacune de ces arêtes à la page qui l'exploite.

## Énumérer les ACE

BloodHound est l'outil pratique car il résout les chemins d'ACL *transitifs* (A peut écrire sur B, B est admin de C) plutôt qu'un objet à la fois ; collectez avec les ACL activées (voir [BloodHound](bloodhound.md)) et lancez les requêtes de droits dangereux. Pour des lectures ciblées :

```
# PowerView: who has rights over a specific object, resolved to names
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs |
  ? { $_.ActiveDirectoryRights -match 'GenericAll|WriteDacl|WriteOwner|GenericWrite' }

# Rights a principal you control holds across the domain
Get-DomainObjectAcl -ResolveGUIDs |
  ? { $_.SecurityIdentifier -eq '<your-SID>' }
```

```bash
# From Linux
nxc ldap <dc> -u user -p pass -M daclread -o TARGET=krbtgt
dacledit.py -action read -target 'Domain Admins' example.local/user:pass
bloodyAD --host <dc> -d example.local -u user -p pass get writable      # objects you can write
```

`-ResolveGUIDs` (et son équivalent) est important : les droits étendus et les écritures de propriétés sont identifiés par des GUID de schéma, donc sans résolution vous ne pouvez pas distinguer `DS-Replication-Get-Changes` (DCSync) d'une écriture de propriété anodine.

## Notes d'exploitation

- Énumérez du point de vue de **chaque** principal que vous contrôlez, y compris les groupes dont vous êtes membre et les comptes machine que vous possédez ; l'ACE exploitable se trouve souvent sur un groupe, pas directement sur votre utilisateur.
- La lecture de la DACL est l'énumération ; transformer chaque ACE en contrôle (réinitialisation, ajout de membre, shadow credentials, DCSync) est traité par les pages de technique de ce sujet, référencées depuis la [vue d'ensemble des DACL](index.md).
- Portez une attention particulière aux ACE sur `AdminSDHolder`, la racine du domaine, le compte `krbtgt` et les Domain/Enterprise Admins, où une seule ACE accessible en écriture est critique pour le domaine.

## Outils

- **BloodHound** : analyse de chemins d'ACL transitifs.
- **PowerView `Get-DomainObjectAcl`** : lectures de DACL en local avec résolution des GUID.
- **dacledit.py** (Impacket) / **NetExec daclread** : lectures de DACL depuis Linux.

## Références

- The Hacker Recipes: DACL abuse
- Microsoft: Active Directory access rights and ACEs
