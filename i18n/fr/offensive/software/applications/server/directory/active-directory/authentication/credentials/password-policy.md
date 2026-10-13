---
order: 8
title: "Énumération de la politique de mots de passe : des seuils pour un spraying sûr"
description: "Lire la politique de mots de passe du domaine Active Directory et les politiques de mots de passe affinées pour connaître les seuils de verrouillage et les fenêtres d'observation avant un password spraying, afin d'éviter les verrouillages de comptes."
keywords:
  - politique de mots de passe
  - seuil de verrouillage
  - politique de mots de passe affinée
  - spraying
  - badPwdCount
---

# Politique de mots de passe

Avant toute tentative de devinette de mot de passe, lisez la politique de verrouillage. Un spraying qui ignore le seuil de verrouillage verrouille des comptes, ce qui est bruyant, perturbateur et souvent une violation de périmètre. La politique vous indique combien de tentatives sont sûres et combien de temps attendre entre les tours.

## Lire la politique de domaine par défaut

La politique de domaine réside sur l'objet domaine et est lisible par tout utilisateur authentifié (et parfois de façon anonyme) :

```bash
nxc smb <dc> -u user -p pass --pass-pol
ldapsearch ... -b 'DC=example,DC=local' -s base \
  lockoutThreshold lockoutDuration lockOutObservationWindow \
  minPwdLength maxPwdAge pwdHistoryLength pwdProperties
```

```
# From a domain host
net accounts /domain
Get-DomainPolicy | Select -Expand SystemAccess
```

Les valeurs qui régissent le spraying :

- **`lockoutThreshold`** : nombre de tentatives erronées avant verrouillage (0 signifie pas de verrouillage, spray à volonté).
- **`lockOutObservationWindow`** : délai avant que le compteur de mauvais mots de passe ne se réinitialise. Spray une tentative par compte, puis attendez un peu plus longtemps que cette fenêtre avant la suivante.
- **`lockoutDuration`** : durée pendant laquelle un compte verrouillé reste verrouillé.

Ces valeurs sont stockées sous forme d'intervalles négatifs de 100 nanosecondes, il faut donc les convertir ; un `lockOutObservationWindow` de `-18000000000` correspond à 30 minutes.

## Politiques de mots de passe affinées

Un domaine peut définir des **politiques de mots de passe affinées** (FGPP, stockées sous forme d'objets `msDS-PasswordSettings` dans le Password Settings Container) qui remplacent la politique par défaut pour des utilisateurs ou groupes spécifiques, souvent avec un verrouillage *plus strict* pour les administrateurs ou des réglages *plus permissifs* pour les comptes de service. Énumérez-les afin qu'une politique propre à un compte ne vous surprenne pas :

```
Get-DomainObject -SearchBase 'CN=Password Settings Container,CN=System,DC=example,DC=local'
```

## Spraying en toute sécurité

La politique étant connue :

- Maintenez le nombre de tentatives par compte et par fenêtre d'observation **en dessous** de `lockoutThreshold`, typiquement une tentative par fenêtre pour être prudent.
- Lisez le `badPwdCount` actuel de chaque compte avant le spraying afin de ne pas faire basculer un compte déjà proche du seuil au-delà de la limite.
- Préférez quand c'est possible un chemin connu pour être sans risque de verrouillage : l'AS-REP roasting et la validation de la pré-authentification Kerberos n'incrémentent jamais le compteur de mauvais mots de passe.

## Notes d'exploitation

- `badPwdCount` n'est **pas répliqué** entre les DC, donc un opérateur rigoureux spray via un seul DC pour conserver un décompte par compte exact.
- Un `lockoutThreshold` de 0 est courant dans les labs et certains domaines de production et signifie que le spraying ne comporte aucun risque de verrouillage, seulement un risque de détection.
- La lecture de la politique relève de l'énumération ; le spraying lui-même est traité sous le bruteforce d'identifiants dans la section authentification et identifiants.

## Outils

- **NetExec (nxc) --pass-pol** : lecture de la politique par défaut.
- **ldapsearch** : attributs bruts de la politique, y compris les objets FGPP.
- **net accounts / PowerView** : lecture de la politique sur l'hôte.

## Références

- [NetExec: --pass-pol](https://github.com/Pennyw0rth/NetExec)
- [Microsoft: Account Policies (password, lockout, Kerberos)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/account-policies)
- [enum4linux-ng (cddmp)](https://github.com/cddmp/enum4linux-ng)
