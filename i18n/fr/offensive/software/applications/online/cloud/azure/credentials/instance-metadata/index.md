---
title: "Métadonnées d'instance"
order: 1
description: "Lire le service de métadonnées d'instance Azure pour générer des jetons d'identité managée, y compris via SSRF contre une ressource victime."
keywords:
  - IMDS
  - métadonnées d'instance
  - jeton d'identité managée
  - SSRF
  - 169.254.169.254
---

# Métadonnées d'instance

Le service de métadonnées d'instance Azure (IMDS) répond sur l'adresse link-local `169.254.169.254` à tout processus d'une VM, et son point de terminaison de jeton distribue un **jeton d'accès pour l'identité managée de la ressource**. Tout point d'ancrage sur une ressource portant une identité managée, exécution de code directe ou server-side request forgery, se transforme en jeton pour tout ce que cette identité peut atteindre.

## Ce qui s'y rattache

- **[Jeton d'identité managée](managed-identity-token.md)** : demander et rejouer le jeton depuis le point de terminaison IMDS sur l'hôte.
- **[SSRF vers IMDS](ssrf-to-imds.md)** : atteindre le même point de terminaison via une application vulnérable.

Contrairement à AWS, l'IMDS Azure exige un en-tête de requête `Metadata: true` et prend un paramètre `resource` nommant l'audience (ARM, Microsoft Graph, Key Vault, stockage), de sorte qu'un jeton est limité à une audience à la fois.

## Références

- [Microsoft : obtenir un jeton d'accès depuis IMDS](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token)
- [HackTricks Cloud : IMDS Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
