---
order: 7
title: "Énumération des utilisateurs et des groupes : constituer la liste des principaux"
description: "Énumérer les utilisateurs et les groupes Active Directory, des lectures LDAP authentifiées au RID cycling non authentifié et à la validation de noms d'utilisateur via Kerberos, pour constituer la liste de cibles du roasting et du spraying."
keywords:
  - énumération des utilisateurs
  - RID cycling
  - kerbrute
  - appartenance aux groupes
  - enum4linux
---

# Énumération des utilisateurs et des groupes

La liste des utilisateurs et des groupes constitue l'ensemble de cibles de presque tout ce qui suit : qui sprayer, qui roaster, qui est privilégié, et quels comptes sont obsolètes. La façon dont vous la construisez dépend de l'accès dont vous disposez, d'une lecture authentifiée complète jusqu'à deviner des noms d'utilisateur sans authentification.

## Énumération authentifiée

Avec n'importe quels identifiants de domaine, lisez les utilisateurs et les groupes via LDAP ou l'interface SAMR :

```bash
nxc smb <dc> -u user -p pass --users          # SAMR user list with lastLogon, badpwd count
nxc ldap <dc> -u user -p pass --groups
net rpc group members "Domain Admins" -U 'EXAMPLE\user%pass' -S <dc>
```

Depuis un hôte du domaine, PowerView et les commandes intégrées donnent la même chose :

```
Get-DomainUser -Properties samaccountname,description,lastlogon
Get-DomainGroupMember "Domain Admins" -Recurse
net group "Domain Admins" /domain
```

Récupérez `description`, `lastLogon`, `pwdLastSet` et `adminCount` en plus des noms : les descriptions laissent fuiter des mots de passe, les comptes obsolètes sont des candidats à mots de passe faibles, et `adminCount=1` signale les comptes actuellement ou anciennement privilégiés (le mécanisme AdminSDHolder marque ainsi les principaux protégés).

## Découverte non authentifiée

Sans identifiants, il existe tout de même des moyens de constituer une liste d'utilisateurs :

- **RID cycling** : là où une session SMB null ou guest est autorisée, parcourez l'espace des RID pour résoudre les SID en noms :

  ```bash
  nxc smb <dc> -u '' -p '' --rid-brute
  enum4linux-ng -R <dc-ip>
  lookupsid.py anonymous@<dc-ip>
  ```

- **Validation de noms d'utilisateur Kerberos** : la réponse de pré-authentification de l'AS-REQ diffère pour les noms d'utilisateur valides et invalides, de sorte qu'une wordlist peut être validée sans une seule tentative de connexion (aucun risque de verrouillage) :

  ```bash
  kerbrute userenum -d example.local --dc <dc-ip> users.txt
  ```

- **Sources externes** : l'OSINT (format d'e-mail de l'entreprise, LinkedIn) amorce la wordlist ; les noms récupérés combinés au format d'e-mail du domaine produisent des `sAMAccountName` candidats.

## Notes d'exploitation

- `kerbrute userenum` est le validateur en masse le plus sûr car il ne soumet jamais de mot de passe, il n'incrémente donc pas le compteur de mauvais mots de passe ni ne déclenche de verrouillage.
- Résolvez l'appartenance aux groupes de façon **récursive** : l'accès privilégié est souvent hérité via des groupes imbriqués plutôt que par une appartenance directe.
- Alimentez la [découverte de SPN](../kerberos/spn-discovery.md) et le password spraying avec la liste d'utilisateurs validée, et [BloodHound](../../dacl/bloodhound.md) avec les comptes privilégiés en tant que cibles de grande valeur.

## Outils

- **NetExec (nxc)** : lectures d'utilisateurs et de groupes SAMR/LDAP, RID brute.
- **kerbrute** : validation de noms d'utilisateur sans verrouillage via Kerberos.
- **enum4linux-ng / lookupsid.py** : RID cycling en session null.
- **PowerView** : requêtes authentifiées sur les utilisateurs, groupes et appartenances.

## Références

- [NetExec: users, groups, and RID brute](https://github.com/Pennyw0rth/NetExec)
- [windapsearch (ropnop)](https://github.com/ropnop/windapsearch)
- [ldapdomaindump (dirkjanm)](https://github.com/dirkjanm/ldapdomaindump)
- [enum4linux-ng (cddmp)](https://github.com/cddmp/enum4linux-ng)
