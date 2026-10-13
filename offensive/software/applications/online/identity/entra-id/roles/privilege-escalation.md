---
title: "Privilege escalation: directory roles that chain to Global Admin"
order: 1
description: "Escalating through Entra directory roles: abusing Privileged Role Administrator, Application Administrator, and similar roles to grant roles, reset credentials, or add app secrets."
keywords:
  - Entra privilege escalation
  - Privileged Role Administrator
  - Application Administrator
  - role assignment
  - Global Administrator
---

# Privilege escalation

Several Entra roles short of Global Administrator reach it anyway. **Privileged Role Administrator** can assign any role, including Global Admin, to anyone. **Application / Cloud Application Administrator** can add credentials to any service principal, then act as an app that holds privileged Graph roles. **Privileged Authentication Administrator** can reset a Global Admin's credentials and MFA. Each is a one-step chain to tenant control.

## Assign yourself a role

```bash
# Privileged Role Administrator: grant Global Administrator to a principal you hold
az rest --method POST \
  --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignments" \
  --body '{"principalId":"<you>","roleDefinitionId":"62e90394-69f5-4237-9190-012177145e10","directoryScopeId":"/"}'
```

```bash
# Application Administrator: add a secret to a privileged app, then log in as it
az ad app credential reset --id <privileged-appId> --append
```

## Exploitation notes

- The Global Administrator roleDefinitionId is `62e90394-69f5-4237-9190-012177145e10`; assigning it at scope `/` is full takeover.
- A Global Admin can cross into Azure by toggling `elevateAccess` to seize User Access Administrator over every subscription, bridging to the [Azure resource plane](../../../../online/cloud/azure/identity/privilege-escalation/elevate-access.md).
- Prefer the quietest chain: adding an app secret (Application Administrator) blends into normal app management better than a direct role grant.

## Tools

- **az cli** / **Graph** (`roleManagement/directory/roleAssignments`).
- **AzureHound** / **BARK**: find role edges that reach Global Admin.

## References

- [SpecterOps: BARK role escalation](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud: Entra role privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Entra built-in roles](https://learn.microsoft.com/entra/identity/role-based-access-control/permissions-reference)
