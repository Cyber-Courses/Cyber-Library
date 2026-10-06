---
title: "Group ownership: an owner adds itself as a member"
description: "Abusing group ownership to add yourself or a controlled principal as a member, inheriting any access or role the group confers."
keywords:
  - group ownership
  - owner
  - add member
  - membership
  - privilege escalation
---

# Group ownership

A group **owner** can manage its membership, so owning a group means you can add yourself to it and inherit whatever it grants: application access, resource roles, or a bound directory role. Ownership is delegated casually and seldom audited, which makes it a common quiet edge into a privileged group.

## Add yourself as a member

```bash
# list groups you own, then add a member
az rest --method GET --url "https://graph.microsoft.com/v1.0/me/ownedObjects"
az rest --method POST --url "https://graph.microsoft.com/v1.0/groups/<group>/members/\$ref" \
  --body '{"@odata.id":"https://graph.microsoft.com/v1.0/directoryObjects/<you>"}'
```

## Exploitation notes

- Target owners of groups that back privileged app roles or that are [role-assignable](role-assignable-groups.md); membership then becomes those permissions.
- Owners can also add a service principal you control, giving a non-interactive foothold in the group.
- Group membership changes are visible in membership listings, so this is more durable than stealthy.

## Tools

- **az cli** / **Graph** (`groups/{id}/members/$ref`).
- **AzureHound** / **BARK**: owner-to-group edges.

## References

- [SpecterOps: AzureHound owner edges](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud: group ownership](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: manage group membership](https://learn.microsoft.com/entra/fundamentals/how-to-manage-groups)
