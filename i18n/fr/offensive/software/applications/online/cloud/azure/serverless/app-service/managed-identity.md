---
title: "Identité managée : forger le jeton App Service"
order: 3
description: "Forger le jeton d'identité managée App Service depuis l'endpoint d'identité local de l'application."
keywords:
  - App Service managed identity
  - identity endpoint
  - token
  - IDENTITY_ENDPOINT
---

# Identité managée

Une application App Service dotée d'une identité managée expose un endpoint de jeton local via `IDENTITY_ENDPOINT` et `IDENTITY_HEADER`, accessible uniquement depuis l'intérieur de l'application. L'exécution de code sur l'application, via [Kudu](kudu-and-scm.md) ou un [web shell](deployment-credentials.md) déployé, forge le jeton de l'identité et poursuit en tant que ce principal auprès d'ARM, de Key Vault ou de Graph.

## Forger le jeton

```bash
# run inside the app (Kudu console or deployed handler)
curl -s "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2019-08-01" \
  -H "X-IDENTITY-HEADER: $IDENTITY_HEADER"
# change resource to https://vault.azure.net or https://graph.microsoft.com to pivot
```

## Notes d'exploitation

- L'endpoint n'est pas l'IMDS de la VM (`169.254.169.254`) ; App Service injecte son propre endpoint et son propre en-tête par application, de sorte que les deux variables d'environnement sont nécessaires.
- Les attributions de rôle de l'identité déterminent le rayon d'impact ; énumérez-les et enchaînez vers l'[identité](../../identity/managed-identities/index.md).
- Une identité affectée par l'utilisateur liée à plusieurs applications transforme la compromission d'une seule application en une identité réutilisée ailleurs.

## Outils

- **curl** contre `IDENTITY_ENDPOINT` depuis le contexte de l'application.
- **MicroBurst** (`Get-AzPasswords`) : extrait les jetons d'identité App Service.

## Références

- [Microsoft: managed identity in App Service](https://learn.microsoft.com/azure/app-service/overview-managed-identity)
- [HackTricks Cloud: Azure managed identities](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps: Managed Identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
