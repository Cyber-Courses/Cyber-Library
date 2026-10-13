---
title: "Accès inter-tenant : faire confiance aux revendications d'un autre tenant"
order: 2
description: "Abus des paramètres d'accès inter-tenant et de la confiance B2B : confiance entrante des revendications de MFA et d'appareil conforme, et synchronisation inter-tenant."
keywords:
  - accès inter-tenant
  - confiance B2B
  - confiance entrante
  - synchronisation inter-tenant
  - multi-tenant
---

# Accès inter-tenant

Les paramètres d'accès inter-tenant déterminent comment un tenant fait confiance aux identités et aux revendications provenant d'un autre. Lorsqu'un tenant accorde une **confiance entrante** aux revendications de MFA ou d'appareil conforme d'un autre tenant, la compromission du tenant de confiance vous permet de satisfaire l'accès conditionnel de la cible avec des revendications forgées dans le tenant que vous contrôlez. La synchronisation inter-tenant peut même provisionner des comptes d'un tenant vers un autre.

## Abuser de la confiance

```bash
# read the target's cross-tenant access policy (as an authenticated principal)
az rest --method GET --url "https://graph.microsoft.com/v1.0/policies/crossTenantAccessPolicy/partners"
# if inbound trust accepts the home tenant's MFA/compliant-device claims, a token from the
# controlled tenant satisfies the target's device/MFA conditional access
```

## Notes d'exploitation

- La confiance entrante du MFA signifie que vous ne refaites pas de MFA depuis le tenant de confiance : prenez le contrôle de ce tenant et ses revendications passent dans la cible.
- La synchronisation inter-tenant provisionne des utilisateurs d'un tenant source vers la cible, une voie vers un compte permanent si vous contrôlez la source.
- Le B2B direct connect partage des ressources (par exemple les canaux partagés Teams) à travers la relation de confiance, élargissant les données accessibles.

## Outils

- **az cli** / **Graph** (`crossTenantAccessPolicy`).
- **ROADtools** : énumération des politiques inter-tenant.

## Références

- [dirkjanm.io : recherche sur la confiance inter-tenant](https://dirkjanm.io/)
- [HackTricks Cloud : accès inter-tenant](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : paramètres d'accès inter-tenant](https://learn.microsoft.com/entra/external-id/cross-tenant-access-overview)
