---
title: "Jonction hybride : forger une identité de périphérique à jonction hybride"
order: 3
description: "Forger une jonction hybride Azure AD : écrire des objets périphérique ou abuser de la synchronisation on-prem pour créer une identité de périphérique à jonction hybride de confiance."
keywords:
  - hybrid join
  - hybrid Azure AD join
  - device object
  - on-prem sync
  - PRT
---

# Jonction hybride

Un périphérique à jonction hybride est un ordinateur Active Directory on-prem qui est également enregistré dans Entra. Avec le contrôle de l'AD on-prem (ou du chemin de synchronisation), vous forgez les éléments d'une jonction hybride, en écrivant l'attribut `userCertificate` du périphérique et en laissant Entra Connect le synchroniser, pour créer une identité à jonction hybride de confiance qui produit un jeton d'actualisation primaire.

## Forger la jonction

```powershell
# write a self-signed device certificate to the on-prem computer object, then let sync carry it to Entra
# AADInternals handles the device cert and PRT once the object is synced
Set-AADIntDeviceRegKeys ; Get-AADIntUserPRTToken
```

Voir [identité hybride](../../../../server/directory/active-directory/trusts/entra-hybrid.md) pour les rouages internes de la synchronisation Entra Connect dont cette technique abuse.

## Notes d'exploitation

- Cela fait le pont entre une compromission de l'AD on-prem et Entra : une identité de périphérique hybride est approuvée par l'accès conditionnel du cloud et produit un PRT cloud.
- Cela nécessite un accès en écriture à l'objet ordinateur on-prem (ou au compte de synchronisation) ; c'est donc un pivot du on-prem vers le cloud, pas une technique externe.
- Le périphérique forgé semble légitimement à jonction hybride, ce qui le rend durable et peu détectable.

## Outils

- **AADInternals** : gestion du certificat de périphérique et du PRT.
- **ROADtools** (`roadtx`) : opérations sur les périphériques et les PRT.

## Références

- [dirkjanm.io : jonction hybride et PRT](https://dirkjanm.io/)
- [AADInternals : jonction hybride](https://aadinternals.com/aadinternals/)
- [HackTricks Cloud : identité hybride](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
