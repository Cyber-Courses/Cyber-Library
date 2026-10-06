---
title: "PIM: activating eligible privileged roles"
order: 4
description: "Activating eligible privileged Azure resource roles through Privileged Identity Management and abusing weak activation or approval settings."
keywords:
  - PIM
  - Privileged Identity Management
  - eligible role
  - activation
  - Azure resource roles
---

# PIM

Privileged Identity Management makes a role **eligible** rather than permanently active: the principal activates it on demand, sometimes behind MFA or an approval. Where a principal you control is eligible for a privileged Azure resource role, activation is a self-service step to the role, and weak activation settings (no approval, no MFA, long windows) make it a reliable escalation.

## Activating an eligible role

```bash
# list roles you are eligible for (resource roles)
az rest --method get --url "https://management.azure.com/providers/Microsoft.Authorization/roleEligibilityScheduleInstances?api-version=2020-10-01&\$filter=asTarget()"

# activate an eligible assignment (self-activation request)
az rest --method put \
  --url "https://management.azure.com/<scope>/providers/Microsoft.Authorization/roleAssignmentScheduleRequests/<guid>?api-version=2020-10-01" \
  --body '{"properties":{"principalId":"<you>","roleDefinitionId":"<roleDefId>","requestType":"SelfActivate","scheduleInfo":{"startDateTime":null,"expiration":{"type":"AfterDuration","duration":"PT8H"}}}}'
```

## Exploitation notes

- Eligibility is itself a target: holding a write over PIM assignments lets you make yourself eligible, then activate.
- Activation without approval or MFA is immediate; where approval is required, the role is only as safe as the approver.
- Azure resource-role PIM lives here; PIM for Entra directory roles is a tenant-plane technique under Entra ID in the Directory area.

## Tools

- **az cli** / `az rest` against the PIM REST API.
- **ROADtools** / **AzureHound**: surface eligible assignments in the graph.

## References

- [Microsoft: activate Azure resource roles in PIM](https://learn.microsoft.com/entra/id-governance/privileged-identity-management/pim-resource-roles-activate-your-roles)
- [SpecterOps: PIM and Azure attack paths](https://posts.specterops.io/)
- [HackTricks Cloud: Azure PIM](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
