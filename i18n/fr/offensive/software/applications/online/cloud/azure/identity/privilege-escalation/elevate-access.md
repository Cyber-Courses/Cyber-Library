---
title: "Elevate access : User Access Administrator au scope racine"
order: 3
description: "Activer User Access Administrator au scope racine avec elevateAccess pour obtenir un contrôle à l'échelle du groupe d'administration sur chaque abonnement du tenant."
keywords:
  - elevateAccess
  - User Access Administrator
  - scope racine
  - groupe d'administration
  - tenant
---

# Elevate access

Un Global Administrator du tenant (un rôle Entra) peut actionner un seul interrupteur qui accorde le rôle **User Access Administrator** au scope **racine** (`/`), au-dessus de chaque groupe d'administration et abonnement. Une fois élevé, le principal peut lire et écrire les attributions de rôle sur tout le tenant, ce qui constitue le pont entre le contrôle de l'annuaire et le contrôle total du plan de ressources.

## L'actionner

```bash
# requires the Global Administrator directory role; grants UAA at "/"
az rest --method post \
  --url "https://management.azure.com/providers/Microsoft.Authorization/elevateAccess?api-version=2016-07-01"

# now assignable at the root scope
az role assignment create --assignee <object-id> --role Owner --scope "/"
```

## Notes d'exploitation

- C'est le seul endroit où le plan d'annuaire Entra et le plan de ressources Azure se rencontrent : il faut un rôle d'annuaire (Global Admin) pour l'effectuer, et il confère une puissance du plan de ressources partout. Les attaques côté annuaire qui mènent à Global Admin se trouvent sous Entra ID dans la section Annuaire.
- L'élévation est journalisée et l'attribution UAA à la racine est visible par quiconque audite le RBAC au scope racine ; elle est bruyante mais totale.
- Retirez l'attribution à la racine par la suite pour réduire l'empreinte tout en conservant les attributions Owner restreintes que vous avez faites.

## Outils

- **az cli** (`az rest` pour le POST elevateAccess).
- **MicroBurst** / **BARK** : automatiser la séquence élévation-puis-attribution.

## Références

- [Microsoft : élever l'accès pour gérer tous les abonnements](https://learn.microsoft.com/azure/role-based-access-control/elevate-access-global-admin)
- [SpecterOps : de Global Admin à Azure avec BARK](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud : élévation de privilèges Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
