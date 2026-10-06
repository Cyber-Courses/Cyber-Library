---
title: "Entra groups"
description: "Taking over Entra security groups to inherit their access: dynamic-membership rule injection, group ownership, and role-assignable groups that grant directory roles."
keywords:
  - Entra groups
  - dynamic membership
  - group ownership
  - role-assignable groups
  - membership
---

# Groups

Entra groups gate access to applications, resources, and even directory roles, so becoming a member of the right group inherits its power without touching the role or resource directly. The ways in are joining a dynamic group by matching its rule, abusing ownership to add yourself, and targeting groups that are bound to directory roles.

## What folds in here

- **[Dynamic membership](dynamic-membership.md)**: setting the attribute a dynamic group's rule matches.
- **[Group ownership](group-ownership.md)**: an owner adding itself as a member.
- **[Role-assignable groups](role-assignable-groups.md)**: groups whose membership grants a directory role.

## References

- [SpecterOps: AzureHound group edges](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud: Entra groups](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Entra groups and membership](https://learn.microsoft.com/entra/identity/users/groups-create-rule)
