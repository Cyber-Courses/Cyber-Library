---
title: "Tenant Entra"
order: 6
description: "Franchir les frontières des tenants Entra : accès invité B2B et politiques d'accès inter-tenant qui exposent des ressources à des identités externes."
keywords:
  - tenant
  - B2B
  - accès invité
  - accès inter-tenant
  - identité externe
---

# Tenant

La frontière du tenant est censée séparer les organisations, mais la collaboration B2B et les paramètres d'accès inter-tenant y percent des trous. Un compte invité est un point d'ancrage à l'intérieur d'un tenant auquel vous êtes externe, et une confiance inter-tenant trop laxiste peut laisser des revendications (MFA, conformité d'appareil) ou une synchronisation passer d'un tenant à un autre.

## Ce qui est regroupé ici

- **[Accès invité](guest-access.md)** : invitation B2B, énumération par un invité et escalade d'invité à membre.
- **[Accès inter-tenant](cross-tenant-access.md)** : confiance entrante et abus de la synchronisation inter-tenant.

## Références

- [dirkjanm.io : recherche inter-tenant et B2B](https://dirkjanm.io/)
- [HackTricks Cloud : tenant Entra](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : paramètres d'accès inter-tenant](https://learn.microsoft.com/entra/external-id/cross-tenant-access-overview)
