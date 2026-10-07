---
title: "Enumeration: mapping role assignments, custom roles, and managed identities"
order: 1
description: "Mapping an Azure subscription's role assignments, custom roles, managed identities, and resource inventory with Azure Resource Graph, az cli, and ROADtools."
keywords:
  - Azure enumeration
  - Resource Graph
  - az cli
  - ROADtools
  - role assignments
  - subscription
---

# Enumeration

Every Azure escalation starts from the same questions: what scopes can my principal see, which roles is it assigned, and which resources carry a managed identity it could borrow. The answers come from the ARM control plane, so enumeration is a set of authenticated reads against Resource Manager and the resource graph.

## Who am I and what can I see

```bash
az account show                      # tenant, subscription, user/SP
az account list --all -o table       # every subscription in reach
az ad signed-in-user show            # the current user object (if a user)
```

## Role assignments and custom roles

```bash
# every assignment visible to you, across the subscription
az role assignment list --all --include-inherited -o table
# custom role definitions (look for wildcard actions)
az role definition list --custom-role-only true --query "[].{name:roleName,actions:permissions[0].actions}"
```

## Resources and managed identities at scale

```bash
# Resource Graph scales across subscriptions in one query
az graph query -q "Resources | project name, type, identity, subscriptionId" --first 1000
# resources carrying an identity are escalation targets
az graph query -q "Resources | where isnotnull(identity) | project name, type, identity"
```

## Graphing the tenant

ROADtools and AzureHound pull the directory and RBAC graph for offline analysis:

```bash
roadrecon auth -u user@tenant -p pass ; roadrecon gather ; roadrecon gui
azurehound -u user@tenant -p pass list --tenant <tenant> -o output.json   # into BloodHound
```

## Exploitation notes

- Resource Graph (`az graph query`) is the fast path: one query inventories resources, identities, and types across every subscription you can read, instead of walking resource groups.
- A resource with `identity` set is a candidate for [managed-identity](managed-identities/index.md) token theft; cross-reference its RBAC assignments to see if the identity is privileged.
- Reader at a management-group scope exposes the whole hierarchy below it, which is often broader than operators expect.

## Tools

- **az cli** (`az role assignment list`, `az graph query`): native enumeration.
- **ROADtools** (`roadrecon`): directory and RBAC graph, offline GUI.
- **AzureHound** / **BARK**: RBAC and resource graph into BloodHound for path analysis.
- **Stormspotter**: Azure attack-surface graph.

## References

- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
- [SpecterOps: AzureHound and BARK](https://github.com/BloodHoundAD/BARK)
- [Microsoft: Azure Resource Graph query language](https://learn.microsoft.com/azure/governance/resource-graph/)
- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
