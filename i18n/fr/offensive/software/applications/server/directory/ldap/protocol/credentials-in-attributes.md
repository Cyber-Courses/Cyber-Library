---
title: "Identifiants dans les attributs : mots de passe et secrets stockés dans les objets de l'annuaire"
description: "Récolter les identifiants que les administrateurs et l'annuaire lui-même stockent dans les attributs d'objets LDAP : les champs description et info, userPassword, et les attributs de mot de passe géré LAPS et gMSA d'Active Directory."
keywords:
  - identifiants dans LDAP
  - champ description
  - userPassword
  - LAPS
  - gMSA
---

# Identifiants dans les attributs

Les annuaires finissent par contenir des identifiants en clair et récupérables dans les attributs d'objets, en partie parce que les administrateurs utilisent les champs comme notes et en partie à cause de fonctionnalités qui stockent les mots de passe de machines et de services dans l'annuaire pour que des principaux autorisés les lisent. Lire les bons attributs transforme un accès à l'annuaire en accès à des identifiants.

## Secrets stockés par les administrateurs

La fuite classique est un mot de passe saisi dans un champ de texte libre :

```bash
# Grep the directory for password-like content in notes fields
ldapsearch ... "(|(description=*pass*)(description=*pwd*)(info=*))" sAMAccountName description info

# Broad sweep for any non-empty description across users
ldapsearch ... "(&(objectClass=user)(description=*))" sAMAccountName description
```

`description`, `info`, `comment` et des champs de schéma personnalisés contiennent fréquemment des mots de passe d'onboarding, des mots de passe de comptes de service et des identifiants « temporaires » qui n'ont jamais été supprimés.

## userPassword

Sur les annuaires non-AD (OpenLDAP, 389 DS), l'attribut `userPassword` stocke le mot de passe du compte, souvent sous forme de hash (`{SSHA}`, `{MD5}`) et occasionnellement en clair. Là où les ACL sont mal configurées, il est lisible par un utilisateur bindé ou même anonymement, et les hashs se cassent hors ligne :

```bash
ldapsearch ... "(objectClass=person)" userPassword
```

Dans AD, `userPassword` n'est généralement pas renseigné ou pas lisible, mais `unixUserPassword` et des attributs similaires apparaissent dans les environnements avec intégration de gestion d'identités.

## LAPS et gMSA d'Active Directory

AD stocke les secrets de machines et de services dans l'annuaire pour les principaux dotés d'un accès en lecture :

- **LAPS** : le mot de passe d'administrateur local d'une machine jointe au domaine est stocké dans `ms-MCS-AdmPwd` (LAPS hérité) ou `msLAPS-Password` (Windows LAPS). Tout principal disposant du droit de lecture (souvent un groupe helpdesk) lit le mot de passe d'administrateur local directement depuis l'objet ordinateur.

  ```bash
  nxc ldap <dc> -u user -p pass -M laps
  ldapsearch ... "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd sAMAccountName
  ```

- **gMSA** : le blob de mot de passe d'un group managed service account est dans `msDS-ManagedPassword`, lisible par les principaux listés dans `msDS-GroupMSAMembership`. Les outils calculent le hash NT à partir du blob renvoyé pour le pass-the-hash.

  ```bash
  nxc ldap <dc> -u user -p pass --gmsa
  gMSADumper.py -u user -p pass -d example.local
  ```

Qui peut lire ceux-ci est une question d'[ACL](../../active-directory/dacl/acl-enumeration.md), de sorte qu'énumérer les droits de lecture sur les objets porteurs de LAPS/gMSA vous indique quels secrets sont à portée.

## Notes d'exploitation

- Balayez `description`/`info` tôt ; c'est la source d'identifiants à zéro privilège au plus fort rendement dans de nombreux domaines.
- Un mot de passe LAPS récupéré donne administrateur local sur cette seule machine (LAPS génère aléatoirement par hôte), ce qui reste un point d'appui pour l'extraction d'identifiants et la chasse aux sessions.
- Les hashs gMSA et de comptes machine sont robustes (longs et aléatoires), ils ne se cassent donc pas, mais ils sont directement utilisables pour le pass-the-hash et, pour les comptes machine, pour l'abus de RBCD et de S4U.

## Outils

- **ldapsearch** : balayages d'attributs pour les champs de notes et `userPassword`.
- **NetExec (nxc) ldap -M laps / --gmsa** : extraction LAPS et gMSA.
- **gMSADumper.py** : calculer les hashs NT gMSA à partir du blob de mot de passe géré.

## Références

- The Hacker Recipes: credential dumping (directory-stored secrets)
- Microsoft: LAPS and group managed service accounts
