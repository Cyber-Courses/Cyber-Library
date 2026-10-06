---
title: "PIM: activating eligible privileged roles"
description: "Abusing Privileged Identity Management: activating eligible roles, exploiting weak activation approval, and standing-eligible assignments for just-in-time escalation."
keywords:
  - PIM
  - Privileged Identity Management
  - eligible role
  - activation
  - just-in-time
---

# PIM

Privileged Identity Management makes privileged roles **eligible** rather than permanently active: a principal activates the role just in time. When you compromise an account that is eligible for a powerful role, activation is often all that stands between you and that role, and activation controls (MFA, approval, justification) are frequently weak or unconfigured.

## Activate an eligible role

```bash
# list your eligible role assignments, then self-activate
az rest --method GET --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleEligibilityScheduleInstances?\$filter=principalId eq '<you>'"
az rest --method POST \
  --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignmentScheduleRequests" \
  --body '{"action":"selfActivate","principalId":"<you>","roleDefinitionId":"<role>","directoryScopeId":"/","justification":"x"}'
```

## Exploitation notes

- Eligibility is a standing escalation: a compromised eligible account becomes the role on demand, so eligibility matters as much as active assignment when mapping targets.
- Activation that requires only justification (no MFA, no approval) is self-service escalation; check the policy before assuming a barrier.
- Activations are time-boxed and logged, so operate within the window and expect the activation event in the audit log.

## Tools

- **az cli** / **Graph** (`roleAssignmentScheduleRequests`).
- **AzureHound**: surfaces eligible-role edges.

## References

- [SpecterOps: AzureHound PIM edges](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud: PIM](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: PIM for Entra roles](https://learn.microsoft.com/entra/id-governance/privileged-identity-management/pim-configure)
