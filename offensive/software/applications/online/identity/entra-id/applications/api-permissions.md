---
title: "API permissions: Microsoft Graph app-role abuse"
order: 3
description: "Abusing Microsoft Graph application permissions: leveraging AppRoleAssignment and high-value scopes such as RoleManagement.ReadWrite.Directory to escalate to tenant control."
keywords:
  - API permissions
  - Microsoft Graph
  - app roles
  - AppRoleAssignment
  - RoleManagement
---

# API permissions

Microsoft Graph **application permissions** (app roles) are granted to service principals and run without a signed-in user. A handful are tenant-ending: `RoleManagement.ReadWrite.Directory` lets an app assign directory roles (including Global Admin), and `AppRoleAssignment.ReadWrite.All` lets an app grant itself any other permission.

## Escalate through a privileged app role

```bash
# find SPs holding dangerous app roles
az ad sp list --all --query "[?appRoles]" 
# with AppRoleAssignment.ReadWrite.All on an SP you control, grant it RoleManagement.ReadWrite.Directory
# POST /servicePrincipals/{id}/appRoleAssignments  (Graph)
# then assign a Global Admin directory role to a principal you control
```

## Exploitation notes

- `AppRoleAssignment.ReadWrite.All` is self-escalating: an app with it grants itself anything, so treat it as equivalent to Global Admin.
- `RoleManagement.ReadWrite.Directory` directly assigns directory roles, the cleanest path to tenant takeover from an app.
- These run as the application with no user and no MFA, which is why an over-permissioned SP is such a strong foothold.

## Tools

- **GraphRunner** / **az cli**: enumerate and assign app roles.
- **BARK** (`Get-AzureADRoleAssignment` family): identify dangerous grants.

## References

- [SpecterOps: BARK and Graph permission abuse](https://github.com/BloodHoundAD/BARK)
- [GraphRunner (dafthack)](https://github.com/dafthack/GraphRunner)
- [HackTricks Cloud: Graph permissions](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
