---
title: "Énumération : cartographier les attributions de rôle, les rôles personnalisés et les identités managées"
order: 1
description: "Cartographier les attributions de rôle, les rôles personnalisés, les identités managées et l'inventaire des ressources d'un abonnement Azure avec Azure Resource Graph, az cli et ROADtools."
keywords:
  - énumération Azure
  - Resource Graph
  - az cli
  - ROADtools
  - attributions de rôle
  - abonnement
---

# Énumération

Toute escalade Azure part des mêmes questions : quels scopes mon principal peut-il voir, quels rôles lui sont attribués et quelles ressources portent une identité managée qu'il pourrait emprunter. Les réponses viennent du plan de contrôle ARM, donc l'énumération est un ensemble de lectures authentifiées contre Resource Manager et le resource graph.

## Qui suis-je et que puis-je voir

```bash
az account show                      # tenant, subscription, user/SP
az account list --all -o table       # every subscription in reach
az ad signed-in-user show            # the current user object (if a user)
```

## Attributions de rôle et rôles personnalisés

```bash
# every assignment visible to you, across the subscription
az role assignment list --all --include-inherited -o table
# custom role definitions (look for wildcard actions)
az role definition list --custom-role-only true --query "[].{name:roleName,actions:permissions[0].actions}"
```

## Ressources et identités managées à grande échelle

```bash
# Resource Graph scales across subscriptions in one query
az graph query -q "Resources | project name, type, identity, subscriptionId" --first 1000
# resources carrying an identity are escalation targets
az graph query -q "Resources | where isnotnull(identity) | project name, type, identity"
```

## Cartographier le tenant

ROADtools et AzureHound récupèrent le graphe de l'annuaire et du RBAC pour une analyse hors ligne :

```bash
roadrecon auth -u user@tenant -p pass ; roadrecon gather ; roadrecon gui
azurehound -u user@tenant -p pass list --tenant <tenant> -o output.json   # into BloodHound
```

## Notes d'exploitation

- Resource Graph (`az graph query`) est la voie rapide : une seule requête inventorie les ressources, les identités et les types sur chaque abonnement que vous pouvez lire, au lieu de parcourir les groupes de ressources un à un.
- Une ressource dont `identity` est renseigné est un candidat au vol de token d'[identité managée](managed-identities/index.md) ; recoupez ses attributions RBAC pour voir si l'identité est privilégiée.
- Le rôle Reader au niveau d'un groupe d'administration expose toute la hiérarchie en dessous, ce qui est souvent plus large que ce que les opérateurs imaginent.

## Outils

- **az cli** (`az role assignment list`, `az graph query`) : énumération native.
- **ROADtools** (`roadrecon`) : graphe de l'annuaire et du RBAC, interface hors ligne.
- **AzureHound** / **BARK** : RBAC et resource graph importés dans BloodHound pour l'analyse de chemins.
- **Stormspotter** : graphe de la surface d'attaque Azure.

## Références

- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
- [SpecterOps : AzureHound et BARK](https://github.com/BloodHoundAD/BARK)
- [Microsoft : langage de requête Azure Resource Graph](https://learn.microsoft.com/azure/governance/resource-graph/)
- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
