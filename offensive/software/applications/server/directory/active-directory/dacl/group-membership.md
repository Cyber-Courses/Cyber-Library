---
title: "Group membership: joining a privileged group"
description: "Using AddMember, AddSelf, or GenericWrite over the member attribute of an Active Directory group to add a controlled principal, inheriting the group's rights, including nested-group and primaryGroupID considerations and the Account Operators path."
keywords:
  - AddMember
  - AddSelf
  - group membership
  - primaryGroupID
  - Account Operators
---

# Group membership

Writing a group's **`member`** attribute (the `AddMember` right, the self-only `AddSelf`, or a broader `GenericWrite`/`GenericAll`) lets you add a controlled principal to it and inherit every right the group holds. Against a privileged group this is immediate escalation, and it is often the simplest edge BloodHound surfaces.

## Adding yourself

```bash
# bloodyAD
bloodyAD --host <dc> -d example.local -u user -p pass add groupMember 'Domain Admins' 'user'

# Samba net
net rpc group addmem 'Domain Admins' 'user' -U 'example.local/user%pass' -S <dc>
```

```powershell
# PowerView
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'user'
```

Membership of a high-value group usually takes effect on the next logon/ticket (the new SID is in the PAC), so request a fresh TGT after adding yourself.

## Details that decide whether it works

- **Nested groups**: AD group membership is transitive, so you do not need an edge to the target group itself. A write over any group that is (even indirectly) a member of a privileged group is enough. BloodHound already resolves this.
- **`primaryGroupID` (stealth, not a way in)**: this does **not** grant a new membership. AD only allows setting a user's `primaryGroupID` to a group the user is **already** a member of, and the update then removes the explicit `member` entry (primary-group membership is implicit). So it can only **hide** a membership you already have, keeping a privileged membership out of the group's `member` list after the fact, not join a group you were not in.
- **Account Operators**: members can modify most non-protected groups and accounts but **not** protected groups ([AdminSDHolder](adminsdholder.md)-guarded ones like Domain Admins). It is a common "almost admin" foothold: use it to reach any unprotected account, not the protected ones directly.
- **Builtin vs domain groups**: adding to `Administrators` (builtin, domain-local) grants domain-controller local admin; adding to `Domain Admins` (global) is broader. Pick the group that matches the access you need.

## Exploitation notes

- Membership changes are visible in the group's `member` list and in directory-service audit events, so they are easy to spot; the `primaryGroupID` variant hides from a casual `member` review.
- Remove yourself afterwards to reduce the footprint, unless you want the membership as persistence.
- A write over a group you did not expect to matter can still win through nesting, so trust the BloodHound path over an eyeball of the obvious groups.

## Tools

- **bloodyAD** (`add groupMember`), **Samba `net rpc group addmem`**: add from Linux.
- **PowerView** (`Add-DomainGroupMember`, `Set-DomainObject` for `primaryGroupID`): on-host.
- **BloodHound**: resolves nested membership and the shortest path to a privileged group.

## References

- The Hacker Recipes: AddMember / group abuse
- SpecterOps: BloodHound AddMember edge
