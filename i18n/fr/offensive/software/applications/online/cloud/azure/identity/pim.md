---
title: "PIM : activer des rôles privilégiés éligibles"
order: 4
description: "Activer des rôles de ressource Azure privilégiés éligibles via Privileged Identity Management et abuser de réglages d'activation ou d'approbation faibles."
keywords:
  - PIM
  - Privileged Identity Management
  - rôle éligible
  - activation
  - rôles de ressource Azure
---

# PIM

Privileged Identity Management rend un rôle **éligible** plutôt que définitivement actif : le principal l'active à la demande, parfois derrière un MFA ou une approbation. Là où un principal que vous contrôlez est éligible pour un rôle de ressource Azure privilégié, l'activation est une étape en libre-service vers le rôle, et des réglages d'activation faibles (pas d'approbation, pas de MFA, longues fenêtres) en font une escalade fiable.

## Activer un rôle éligible

```bash
# list roles you are eligible for (resource roles)
az rest --method get --url "https://management.azure.com/providers/Microsoft.Authorization/roleEligibilityScheduleInstances?api-version=2020-10-01&\$filter=asTarget()"

# activate an eligible assignment (self-activation request)
az rest --method put \
  --url "https://management.azure.com/<scope>/providers/Microsoft.Authorization/roleAssignmentScheduleRequests/<guid>?api-version=2020-10-01" \
  --body '{"properties":{"principalId":"<you>","roleDefinitionId":"<roleDefId>","requestType":"SelfActivate","scheduleInfo":{"startDateTime":null,"expiration":{"type":"AfterDuration","duration":"PT8H"}}}}'
```

## Notes d'exploitation

- L'éligibilité est elle-même une cible : détenir une écriture sur les attributions PIM permet de se rendre éligible, puis d'activer.
- L'activation sans approbation ni MFA est immédiate ; lorsque l'approbation est requise, le rôle n'est aussi sûr que l'approbateur.
- Le PIM des rôles de ressource Azure est traité ici ; le PIM des rôles d'annuaire Entra est une technique du plan tenant, sous Entra ID dans la section Annuaire.

## Outils

- **az cli** / `az rest` contre l'API REST PIM.
- **ROADtools** / **AzureHound** : font ressortir les attributions éligibles dans le graphe.

## Références

- [Microsoft : activer des rôles de ressource Azure dans PIM](https://learn.microsoft.com/entra/id-governance/privileged-identity-management/pim-resource-roles-activate-your-roles)
- [SpecterOps : PIM et chemins d'attaque Azure](https://posts.specterops.io/)
- [HackTricks Cloud : Azure PIM](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
