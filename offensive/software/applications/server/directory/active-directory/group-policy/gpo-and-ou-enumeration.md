---
title: "GPO and OU enumeration: Group Policy objects, links, and scope"
order: 1
description: "Enumerating Active Directory Group Policy objects and organizational units: which GPOs exist, where they are linked, who they apply to, and who can edit them, to find policy-based privilege-escalation paths."
keywords:
  - GPO enumeration
  - organizational unit
  - gPLink
  - group policy
  - GPO permissions
---

# GPO and OU enumeration

Group Policy objects (GPOs) push configuration (and, usefully for an attacker, scheduled tasks, scripts, and group membership) to the users and computers they are linked to. A principal who can **edit** a GPO can run code on everything that GPO applies to. Enumeration here answers three questions: what GPOs exist, what they are linked to (so you know the blast radius), and who can modify them.

## Mapping GPOs to their targets

GPOs are linked to sites, domains, and organizational units via the `gPLink` attribute on the container. To know who a GPO affects, map GPO to link to the objects under that container:

```
# PowerView
Get-DomainGPO -Properties displayname,gpcfilesyspath
Get-DomainOU -Properties name,gplink
Get-DomainGPO -Identity '{GUID}' | Get-DomainOU   # resolve a GPO to its linked OUs
Get-DomainComputer -SearchBase 'OU=Workstations,DC=example,DC=local'   # who is under that OU
```

```bash
# From Linux
nxc ldap <dc> -u user -p pass --gpo
ldapsearch ... '(objectClass=groupPolicyContainer)' displayName gPCFileSysPath
```

The `gPCFileSysPath` points at the GPO's files in `SYSVOL` (`\\domain\SYSVOL\domain\Policies\{GUID}`), readable by any domain user, which is where Group Policy Preferences passwords historically leaked.

## Finding editable GPOs

The escalation path is a GPO whose DACL grants a principal you control write access (`WriteProperty`/`GenericWrite`/`GenericAll`). Enumerate GPO permissions the same way as any object ACL:

```
Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs |
  ? { $_.ActiveDirectoryRights -match 'WriteProperty|GenericWrite|GenericAll' }
```

BloodHound draws this as a `GenericWrite`/`GPOAbuse` edge from the principal to the GPO, and then shows every computer and user the GPO reaches.

## Exploitation notes

- Prioritize GPOs linked to OUs that contain **privileged or many** computers; editing a GPO linked to a Domain Controllers OU is domain-critical.
- A readable `SYSVOL` policy tree is worth grepping for `cpassword` and autologon secrets ([Group Policy Preferences](group-policy-preferences.md)) and for scripts referencing credentials.
- Enumeration identifies the editable, high-reach GPO; turning it into execution is covered by [editing a GPO](editing-a-gpo.md), and reaching objects through a writable OU by [linking a GPO](linking-a-gpo.md).

## Tools

- **PowerView `Get-DomainGPO` / `Get-DomainOU`**: GPO and OU mapping with ACLs.
- **NetExec (`nxc`) `ldap --gpo`**: GPO listing from Linux (plus `-M gpp_password` on SYSVOL).
- **Group3r / Grouper2**: audit GPO contents and ACEs for abusable settings.
- **BloodHound**: GPO-to-target reach, `GenericWrite`-over-GPO and `WriteGPLink` edges.

## References

- The Hacker Recipes: Group policies
- Microsoft: Group Policy architecture and gPLink
