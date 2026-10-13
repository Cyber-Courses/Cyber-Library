---
order: 6
title: "Password spraying : deviner des identifiants valides sans verrouillage"
description: "Deviner les mots de passe Active Directory en toute sécurité en essayant un seul ou quelques mots de passe probables contre de nombreux comptes, en respectant la politique de verrouillage et en utilisant des protocoles sans risque de verrouillage comme la pré-authentification Kerberos."
keywords:
  - password spraying
  - verrouillage
  - kerbrute
  - credential stuffing
  - accès initial
---

# Password spraying

Le spraying inverse la boucle du brute-force : au lieu de nombreux mots de passe contre un seul compte (ce qui le verrouille), vous essayez **un** mot de passe probable contre **chaque** compte, puis attendez et essayez le suivant. C'est la façon la plus courante d'obtenir un point d'appui initial dans un domaine depuis une position non authentifiée, et ce n'est sûr que si vous respectez la politique de verrouillage lue durant l'[énumération](password-policy.md).

## Choisir les candidats

Le spraying réussit sur les mots de passe prévisibles et conformes à la politique :

- **Saison et année** : `Autumn2026`, `Winter2026!`, correspondant à la règle de complexité.
- **Nom de l'entreprise suivi d'un suffixe** : `Example123`, `Example2026!`.
- **`Password1`, `Welcome1`, `Changeme123`** et les mots de passe par défaut / d'intégration (onboarding).
- **Le nom d'utilisateur comme mot de passe**, et les mots de passe vides là où `PASSWD_NOTREQD` est défini.

Construisez la liste de comptes à partir de l'[énumération des utilisateurs](user-and-group-enumeration.md).

## Spraying en toute sécurité

```bash
# Kerberos pre-auth spray: fast and quiet, but a wrong password still counts
# toward lockout, so observe the policy and badPwdCount exactly as with SMB
kerbrute passwordspray -d example.local --dc <dc> users.txt 'Autumn2026!'

# NetExec over SMB/LDAP (watch lockout; use --continue-on-success for a full sweep)
nxc smb <dc> -u users.txt -p 'Autumn2026!' --continue-on-success

# Spray one password, then wait out the lockout observation window before the next
```

Les règles qui gardent les comptes déverrouillés :

- **Une tentative par compte et par fenêtre d'observation**, en restant sous `lockoutThreshold`.
- **Lisez d'abord `badPwdCount`** afin de ne pas faire basculer au-delà de la limite un compte déjà proche du seuil.
- **Spray via un seul DC**, car `badPwdCount` n'est pas répliqué entre les DC, de sorte qu'un décompte réparti sur plusieurs DC sous-estime la valeur et risque le verrouillage.

## Éviter le verrouillage et la devinette tout court

Un mauvais mot de passe incrémente `badPwdCount` quel que soit le protocole, pré-authentification Kerberos comprise, il n'existe donc pas de sprayer « gratuit » qui devine sans risque de verrouillage. Ce qui est réellement gratuit, c'est tout ce qui ne soumet pas de mot de passe :

- **Énumération des noms d'utilisateur** (kerbrute `userenum`) : envoie une AS-REQ sans données de pré-authentification et lit si le compte existe, sans jamais toucher à `badPwdCount`. Utilisez-la pour réduire la liste aux comptes valides avant de spray un seul mot de passe.
- **L'AS-REP roasting** contourne entièrement la devinette pour les comptes qui n'exigent pas de pré-authentification, en récupérant un hash crackable sans aucune tentative de connexion (voir la section Kerberos).

Pour la devinette elle-même, la pré-authentification Kerberos est rapide et discrète, mais elle est régie par la même politique de verrouillage que SMB : une tentative par compte et par fenêtre, sous le seuil.

## Notes d'exploitation

- Un seul identifiant valide, aussi peu privilégié soit-il, débloque l'[énumération LDAP](ldap-enumeration.md) complète, le [Kerberoasting](../kerberos/spn-discovery.md) et la collecte BloodHound, de sorte qu'un seul succès transforme la mission.
- Le spraying est bruyant dans les journaux d'authentification même lorsqu'il évite le verrouillage ; rythmez-le et préférez Kerberos pour réduire l'empreinte.
- Lorsqu'un domaine fixe `lockoutThreshold = 0`, le spraying ne comporte aucun risque de verrouillage et peut être plus agressif.

## Outils

- **kerbrute** : spraying rapide via pré-authentification Kerberos, plus énumération des noms d'utilisateur sans verrouillage (`userenum`) pour valider d'abord les comptes.
- **NetExec (nxc)** : spraying sur SMB, LDAP, WinRM, MSSQL, avec prise en compte du verrouillage.
- **Spray / DomainPasswordSpray** : sprayers alternatifs qui lisent d'abord la politique.

## Références

- [kerbrute (ropnop)](https://github.com/ropnop/kerbrute)
- [TREVORspray (blacklanternsecurity)](https://github.com/blacklanternsecurity/TREVORspray)
- [NetExec: password spraying](https://github.com/Pennyw0rth/NetExec)
- [Microsoft: Account Policies (lockout)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/account-policies)
