---
title: "BPRT : jetons d'enrôlement en masse pour enregistrer des périphériques à grande échelle"
order: 2
description: "Abus d'un jeton de provisionnement d'enrôlement en masse pour enregistrer des périphériques et générer des jetons d'actualisation primaires à grande échelle."
keywords:
  - BPRT
  - bulk enrollment
  - provisioning token
  - device registration
  - PRT
---

# BPRT

Un jeton de provisionnement d'enrôlement en masse (BPRT, bulk-enrollment provisioning token) est une information d'identification réutilisable destinée au provisionnement de nombreux périphériques en une seule fois. Obtenu ou forgé, il permet d'enregistrer des périphériques sans jeton propre à chaque utilisateur et de générer des jetons d'actualisation primaires à grande échelle, une primitive puissante de persistance et d'usurpation d'identité.

## Créer et utiliser un BPRT

```powershell
# AADInternals: create a BPRT (needs an account allowed to bulk-enrol), then use it
$bprt = New-AADIntBulkPRTToken -Name "kiosk"
# register a device with the BPRT and obtain a PRT
Join-AADIntDeviceToAzureAD -DeviceName "kiosk-07" -BPRT $bprt
```

## Notes d'exploitation

- Un BPRT n'est pas lié à un seul utilisateur : il permet donc d'enregistrer des périphériques arbitraires et reste réutilisable jusqu'à son expiration ou sa révocation, une excellente persistance.
- Les périphériques créés via un BPRT apparaissent comme légitimement provisionnés, se fondant dans l'enrôlement normal.
- La création d'un BPRT nécessite un compte disposant de droits d'enrôlement en masse ; c'est donc généralement une étape de persistance post-escalade.

## Outils

- **AADInternals** (`New-AADIntBulkPRTToken`).

## Références

- [AADInternals : BPRT](https://aadinternals.com/aadinternals/)
- [dirkjanm.io : identité de périphérique](https://dirkjanm.io/)
- [HackTricks Cloud : abus de périphériques](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
