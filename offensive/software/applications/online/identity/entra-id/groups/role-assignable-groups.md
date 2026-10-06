---
title: "Role-assignable groups: membership grants a directory role"
description: "Targeting role-assignable groups: membership grants the directory roles bound to the group, so group control becomes role control."
keywords:
  - role-assignable groups
  - isAssignableToRole
  - directory role
  - membership
  - privilege escalation
---

# Role-assignable groups

A group created with `isAssignableToRole` can have directory roles assigned to it, and every member inherits those roles. So control of such a group, through [ownership](group-ownership.md), [dynamic membership](dynamic-membership.md), or a role that can edit its members, is control of the roles bound to it, potentially up to Global Administrator.

## Find and join

```bash
# list role-assignable groups
az rest --method GET --url "https://graph.microsoft.com/v1.0/groups?\$filter=isAssignableToRole eq true"
# check which roles are assigned to the group, then get yourself added as a member
az rest --method GET --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignments?\$filter=principalId eq '<group>'"
```

## Exploitation notes

- Role-assignable groups are protected (only higher-privileged roles can manage them), but an owner set at creation or a mis-scoped manager still gets you in.
- A single role-assignable group bound to Global Admin turns any membership edge into tenant takeover.
- Adding a controlled service principal as a member gives a non-interactive, MFA-exempt path to the role.

## Tools

- **az cli** / **Graph** (`groups` isAssignableToRole, `roleAssignments`).
- **AzureHound** / **BARK**: group-to-role edges.

## References

- [SpecterOps: BARK group-to-role](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud: role-assignable groups](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: role-assignable groups](https://learn.microsoft.com/entra/identity/role-based-access-control/groups-concept)
