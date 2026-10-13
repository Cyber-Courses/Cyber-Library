---
order: 15
title: "Comptes d'ordinateur pré-créés : le mot de passe machine par défaut"
description: "Prendre le contrôle d'objets ordinateur pré-préparés dans Active Directory mais jamais joints, dont le mot de passe prend par défaut la valeur du nom d'ordinateur en minuscules, offrant gratuitement un identifiant de compte d'ordinateur authentifié et un chemin vers la délégation contrainte basée sur les ressources ou les shadow credentials."
keywords:
  - ordinateur pré-créé
  - pré-Windows 2000
  - mot de passe de compte machine
  - pre2k
  - PASSWD_NOTREQD
---

# Comptes d'ordinateur pré-créés

Lorsqu'un administrateur pré-prépare un objet ordinateur avec l'option **« attribuer ce compte d'ordinateur en tant qu'ordinateur pré-Windows 2000 »**, Active Directory fixe son mot de passe à une **valeur par défaut prévisible : le nom de l'ordinateur en minuscules**, limité aux 14 premiers caractères, sans le `$` final. (Les objets créés sans cette option reçoivent un mot de passe aléatoire, cela s'applique donc spécifiquement au cas pré-Windows 2000.) Si un tel objet ne s'est jamais authentifié, la valeur par défaut tient généralement encore, ce qui vous met gratuitement entre les mains un **identifiant de compte d'ordinateur** fonctionnel.

## Les trouver

Les candidats portent `PASSWD_NOTREQD` et `WORKSTATION_TRUST_ACCOUNT` dans `userAccountControl` (la valeur combinée `4128`) et un `logonCount` de `0`, ce qui signifie que le compte existe mais ne s'est jamais connecté :

```bash
# pre2k: discover and test likely pre-created computers, authenticating with name-as-password
pre2k unauth -d example.local -dc-ip <dc> -inputfile computers.txt
pre2k auth -d example.local -dc-ip <dc> -u user -p pass
```

```text
# LDAP filter for the tell-tale flags and no prior logon
(&(objectClass=computer)(userAccountControl:1.2.840.113556.1.4.803:=4128)(logonCount=0))
```

## D'un compte pré-créé à la prise de contrôle

Le mot de passe deviné authentifie en tant qu'ordinateur. Réinitialisez-le à une valeur que vous contrôlez, en utilisant le transport RPC ou kpasswd (le protocole `smb-samr` par défaut échoue pour un compte machine ne s'étant jamais connecté) :

```bash
# -protocol rpc-samr (or kpasswd) changes the initial password; the smb-samr default
# fails on an unused computer account with STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT
changepasswd.py -protocol rpc-samr 'example.local/WKSTN01$:wkstn01@<dc>' -newpass 'Password123!'
```

- Le résultat est une **identité d'ordinateur** contrôlée par l'attaquant et porteuse d'un SPN. C'est le compte que vous désignez comme principal délégué dans une chaîne de [délégation contrainte basée sur les ressources](../kerberos/delegation/resource-based-constrained.md), mais la compléter nécessite tout de même une écriture distincte sur l'attribut `msDS-AllowedToActOnBehalfOfOtherIdentity` de la **cible** ; le compte pré-créé fournit le principal contrôlé, pas l'écriture sur la cible.
- À lui seul, il constitue un simple point d'appui authentifié pour l'[énumération LDAP](ldap-enumeration.md) et le [roasting](../kerberos/roasting.md) lorsque vous êtes parti de rien.

## Notes d'exploitation

- Il s'agit d'une **valeur par défaut héritée** qui survit pendant des années : l'option de compatibilité pré-Windows 2000 est antérieure aux jointures modernes, mais les administrateurs pré-préparent encore des machines et laissent en place le mot de passe prévisible.
- `logonCount=0` combiné au flag `PASSWD_NOTREQD` est le signal fort ; une machine qui s'est connectée aura fait tourner son mot de passe.
- Le compte d'ordinateur possédé dispose rarement de droits particuliers par lui-même, sa valeur réside donc dans son rôle de **principal contrôlé pour une chaîne de délégation** (moyennant une écriture distincte sur la cible) et de point d'appui authentifié, pas dans l'accès propre du compte.

## Outils

- **pre2k** (garrettfoster13) : découvrir et s'authentifier aux ordinateurs pré-créés.
- **Impacket** (`changepasswd.py`, `addcomputer.py`) : réinitialiser le mot de passe machine et opérer le compte.

## Références

- [The Hacker Recipes: pre-Windows 2000 computers](https://www.thehacker.recipes/ad/movement/builtins/pre-windows-2000-computers)
- [TrustedSec: diving into pre-created computer accounts](https://trustedsec.com/blog/diving-into-pre-created-computer-accounts)
- [pre2k (garrettfoster13)](https://github.com/garrettfoster13/pre2k)
