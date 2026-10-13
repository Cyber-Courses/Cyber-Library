---
title: "Functions"
order: 1
description: "Détourner Azure Functions : s'exécuter en tant qu'identité managée de l'application et récupérer les function keys et host keys."
keywords:
  - Azure Functions
  - managed identity
  - function keys
  - host key
  - serverless
---

# Functions

Une Function app Azure exécute votre code et, lorsqu'une est configurée, porte une **identité managée**. L'exécution de code dans l'application (via un déploiement, une dépendance ou une injection) forge le jeton de cette identité ; par ailleurs, les **function keys et host keys** de l'application autorisent l'invocation des functions protégées et des endpoints d'administration.

## Ce qui se rattache ici

- **[Identité managée](managed-identity.md)** : forger et utiliser le jeton d'identité managée de la Function app.
- **[Function keys](function-keys.md)** : récupérer les host keys et function keys pour invoquer des endpoints protégés.

## Références

- [HackTricks Cloud: Azure Functions](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Azure Functions access keys](https://learn.microsoft.com/azure/azure-functions/function-keys-how-to)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
