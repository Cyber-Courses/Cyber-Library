---
title: "Identité managée : forger le jeton de la Function app"
order: 1
description: "S'exécuter dans une Azure Function pour forger et utiliser le jeton d'identité managée de la Function app."
keywords:
  - Function managed identity
  - token
  - IMDS
  - privilege escalation
---

# Identité managée

Une Function app dotée d'une **identité managée** affectée par le système ou par l'utilisateur expose un endpoint de jeton local au code en cours d'exécution, via les variables d'environnement `IDENTITY_ENDPOINT` et `IDENTITY_HEADER`. Toute exécution à l'intérieur de la function lit celles-ci et forge un jeton pour ARM, Graph, Key Vault ou le stockage, puis agit en tant que l'identité, dont les attributions de rôle sont fréquemment plus larges que ce que l'application justifie.

## Forger le jeton depuis l'intérieur de la function

```bash
# App Service / Functions managed-identity endpoint (not 169.254.169.254)
curl -s "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2019-08-01" \
  -H "X-IDENTITY-HEADER: $IDENTITY_HEADER"
# swap resource for https://vault.azure.net or https://graph.microsoft.com to pivot
```

Exportez l'`access_token` renvoyé et appelez ARM directement :

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://management.azure.com/subscriptions?api-version=2020-01-01"
```

## Notes d'exploitation

- Pour obtenir l'exécution en premier lieu, déployez ou écrasez du code (voir [identifiants de déploiement](../app-service/deployment-credentials.md) et [Kudu](../app-service/kudu-and-scm.md)), ou utilisez une injection applicative dans le code hébergé.
- La puissance du jeton tient aux attributions de rôle de l'identité ; énumérez-les avec le jeton et enchaînez vers l'[identité](../../identity/managed-identities/index.md).
- Une identité affectée par l'utilisateur partagée entre plusieurs applications signifie qu'une seule function compromise livre une identité utilisée ailleurs.

## Outils

- **curl** contre `IDENTITY_ENDPOINT`.
- **MicroBurst** (`Get-AzPasswords`) : extrait les jetons d'identité managée et les secrets d'application à grande échelle.

## Références

- [HackTricks Cloud: Azure managed identities](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: managed identity token endpoint for App Service and Functions](https://learn.microsoft.com/azure/app-service/overview-managed-identity)
- [SpecterOps: Managed Identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
