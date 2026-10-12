---
title: "Identifiants Azure"
order: 2
description: "Récolter les identifiants Azure : jetons d'identité managée depuis IMDS, secrets et clés Key Vault, clés de compte de stockage, assets des Automation Accounts, et app settings et chaînes de connexion."
keywords:
  - identifiants Azure
  - IMDS
  - Key Vault
  - clés de stockage
  - assets Automation
---

# Identifiants

Chaque appel ARM et de plan de données nécessite un jeton ou une clé, de sorte que les récolter est le moyen d'élargir l'accès. Les identifiants Azure se présentent sous la forme de **jetons d'identité managée** générés depuis le service de métadonnées d'instance, de **jetons d'accès et de rafraîchissement** d'utilisateurs et de services principaux, de **clés de compte de stockage** qui déverrouillent tout le plan de données d'un compte, et des secrets que des services comme Key Vault, les Automation Accounts et App Service renvoient à la lecture.

## Ce qui s'y rattache

- **[Métadonnées d'instance](instance-metadata/index.md)** : générer le jeton d'identité managée d'une ressource depuis IMDS, directement ou via SSRF.
- **[Key Vault](key-vault/index.md)** : secrets, clés et certificats sur le plan de données du coffre.
- **[Clés de stockage](storage-keys.md)** : `listKeys` pour un accès complet au plan de données d'un compte.
- **[Jetons d'accès](access-tokens.md)** : jetons `az` et MSAL en cache pillés sur le disque.
- **[Assets Automation](automation-assets.md)** : assets credential, variable et connexion des Automation Accounts.
- **[App settings et chaînes de connexion](app-settings-and-connection-strings.md)** : secrets de configuration des Function et Web Apps.

Les pages IMDS sont la face Azure de la technique web de [server-side request forgery](../../../../server/web/code/injection/request-forgery/index.md), référencée en renvoi plutôt que dupliquée.

## Références

- [HackTricks Cloud : identifiants Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst (Get-AzPasswords)](https://github.com/NetSPI/MicroBurst)
- [Microsoft : identités managées et IMDS](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token)
