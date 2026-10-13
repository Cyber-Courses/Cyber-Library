---
title: "Assets Automation : lire les secrets en clair des Automation Accounts"
order: 5
description: "Lire les assets de type credential, variable et connexion des Automation Accounts, souvent stockés en clair, pour y trouver des secrets."
keywords:
  - Automation Account
  - asset credential
  - variable
  - connexion
  - secret en clair
---

# Assets Automation

Les Azure Automation Accounts stockent des **assets** réutilisables : des objets credential (nom d'utilisateur et mot de passe), des variables (souvent marquées chiffrées mais lisibles depuis l'intérieur d'un runbook) et des connexions (détails du service principal du compte RunAs). Ce sont un réservoir dense de secrets permanents, et un principal qui peut lire ou exécuter des runbooks les extrait.

## Lire les assets depuis le plan de gestion

```bash
az automation account list -o table
# variables (non-encrypted values return directly)
az rest --method get --url "https://management.azure.com/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Automation/automationAccounts/<aa>/variables?api-version=2023-11-01"
```

## Extraire les assets chiffrés via un runbook

Les variables chiffrées et les assets credential ne sont pas retournés par l'API, mais un runbook s'exécutant à l'intérieur de l'Automation Account peut les lire en clair et les afficher :

```powershell
# publish and start a runbook with this body; it runs with the account's access
$cred = Get-AutomationPSCredential -Name '<credAsset>'
"$($cred.UserName):$($cred.GetNetworkCredential().Password)"
Get-AutomationVariable -Name '<encryptedVar>'
```

## Notes d'exploitation

- Le `Get-AzPasswords` de MicroBurst automatise cela : il publie un runbook temporaire qui déverse chaque asset credential, variable et connexion ainsi que le certificat RunAs.
- L'asset **connexion** RunAs fournit un certificat de service principal, un identifiant durable pour ce principal (voir [Serverless > Automation Accounts](../serverless/automation-accounts/index.md)).
- Écrire et exécuter un runbook exécute aussi du code sous l'identité managée de l'Automation Account, ce qui relie cela à l'[escalade de privilèges](../identity/privilege-escalation/index.md).

## Outils

- **MicroBurst** (`Get-AzPasswords`) : déverse tous les assets Automation et le certificat RunAs.
- **az cli** / **az rest** : lire directement les variables non chiffrées.

## Références

- [NetSPI : MicroBurst et secrets des Automation Accounts](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud : Azure Automation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : assets Automation](https://learn.microsoft.com/azure/automation/shared-resources/credentials)
