---
title: "Enregistrement de périphérique : un périphérique malveillant pour un PRT"
order: 1
description: "Enregistrement d'un périphérique malveillant dans Entra pour obtenir une identité de périphérique et un jeton d'actualisation primaire, satisfaisant l'accès conditionnel basé sur le périphérique."
keywords:
  - device registration
  - Azure AD registered
  - PRT
  - device identity
  - conditional access
---

# Enregistrement de périphérique

Avec le jeton d'un utilisateur, vous pouvez enregistrer un tout nouvel objet périphérique dans Entra sous son identité. Le périphérique enregistré reçoit son propre matériel de clé et un **jeton d'actualisation primaire** (primary refresh token), qui vous confère à la fois un SSO durable en tant que l'utilisateur et fait de vous un périphérique auquel l'accès conditionnel basé sur le périphérique fera confiance.

## Enregistrer un périphérique et obtenir un PRT

```powershell
# AADInternals: join a fake device using the user's token, then mint a PRT
$at = Get-AADIntAccessTokenForAADJoin
Join-AADIntDeviceToAzureAD -DeviceName "laptop-01" -DeviceType "Windows" -AccessToken $at
# use the returned device certificate to request a user PRT
Get-AADIntUserPRTToken
```

```bash
# ROADtools: roadtx registers a device and handles the PRT
roadtx device -a register -n laptop-01
```

## Notes d'exploitation

- Un périphérique enregistré satisfait les stratégies d'accès conditionnel « exiger un périphérique conforme/enregistré », contournant un contrôle courant ([accès conditionnel](../authentication/conditional-access.md)).
- Le PRT du périphérique constitue une persistance durable en tant que l'utilisateur jusqu'à ce que le périphérique soit supprimé.
- L'enregistrement (par opposition à la jonction) ne nécessite généralement qu'un jeton d'utilisateur valide, sans droits d'administration, si l'enregistrement de périphériques est ouvert (ce qui est le cas par défaut dans de nombreux locataires).

## Outils

- **AADInternals** (`Join-AADIntDeviceToAzureAD`).
- **ROADtools** (`roadtx device`).

## Références

- [dirkjanm.io : enregistrement de périphérique et PRT](https://dirkjanm.io/)
- [AADInternals : jonction de périphérique](https://aadinternals.com/aadinternals/)
- [ROADtools](https://github.com/dirkjanm/ROADtools)
