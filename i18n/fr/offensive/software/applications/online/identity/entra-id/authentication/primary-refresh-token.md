---
title: "Primary refresh token : voler et forger le PRT"
order: 4
description: "Voler et rejouer le primary refresh token : extraire le PRT et la clé de session d'un appareil joint et forger des cookies PRT avec ROADtoken et AADInternals."
keywords:
  - primary refresh token
  - PRT
  - cookie PRT
  - ROADtoken
  - AADInternals
---

# Primary refresh token

Le **primary refresh token** est l'identifiant lié à l'appareil qui fournit le single sign-on Entra sur une machine jointe. Avec un accès local à l'appareil, vous extrayez le PRT et sa clé de session, puis vous forgez des **cookies PRT** (`x-ms-RefreshTokenCredential`) qui s'authentifient auprès de n'importe quelle application Entra en tant qu'utilisateur, y compris à travers la MFA, car le PRT porte déjà le claim d'authentification forte.

## Extraire et forger

```powershell
# AADInternals: pull the PRT and session key from the signed-in device
Get-AADIntUserPRTToken
# or derive a PRT from stolen key material
Export-AADIntLocalDeviceCertificate ; New-AADIntUserPRTToken

# ROADtoken: generate a PRT cookie usable in a browser
roadtoken -a <prt> --derived-sessionkey <key>
```

Insérez le cookie résultant dans la requête `login.microsoftonline.com` comme `x-ms-RefreshTokenCredential` et vous êtes connecté en tant qu'utilisateur.

## Notes d'exploitation

- Le PRT satisfait la MFA à lui seul, de sorte qu'un cookie PRT atteint les applications protégées par l'accès conditionnel sans second facteur.
- Les PRT sont liés à l'appareil ; il vous faut la clé de transport de l'appareil, ce qui explique pourquoi l'admin local ou un hook sur le courtier de jetons de la machine est le prérequis.
- Un cookie PRT forgé est un point d'ancrage SSO durable jusqu'à ce que le PRT soit révoqué ou l'appareil désactivé.

## Outils

- **AADInternals** (`Get-AADIntUserPRTToken`) : extraire et générer des jetons PRT.
- **ROADtoken** / **roadtx** (ROADtools) : forger des cookies PRT.

## Références

- [dirkjanm.io : digging into the PRT](https://dirkjanm.io/)
- [AADInternals : PRT](https://aadinternals.com/aadinternals/)
- [ROADtools (roadtx)](https://github.com/dirkjanm/ROADtools)
