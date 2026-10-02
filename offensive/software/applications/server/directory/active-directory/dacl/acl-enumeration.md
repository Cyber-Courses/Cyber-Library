---
title: "ACL enumeration: finding abusable access-control entries"
description: "Enumerating Active Directory object DACLs to find access-control entries a controlled principal can abuse: GenericAll, WriteDacl, WriteOwner, GenericWrite, and the extended rights that lead to privilege escalation."
keywords:
  - ACL enumeration
  - DACL
  - GenericAll
  - WriteDacl
  - extended rights
---

# ACL enumeration

Every AD object carries a discretionary access-control list (DACL) of access-control entries (ACEs) that say which principals may read or modify it. Misconfigured ACEs are the most common privilege-escalation path in AD: a low-privileged principal with the right ACE over a privileged object can reset a password, add itself to a group, or take ownership. Enumeration here is about finding, from what you control, which objects you have dangerous rights over.

## What to look for

The high-value rights, in rough order of power:

- **`GenericAll`**: full control (reset password, add to group, write any attribute).
- **`WriteDacl`**: rewrite the object's DACL, then grant yourself `GenericAll`.
- **`WriteOwner`**: take ownership, then rewrite the DACL.
- **`GenericWrite` / `WriteProperty`**: write specific attributes, `servicePrincipalName` (targeted roasting), `msDS-KeyCredentialLink` (shadow credentials), `msDS-AllowedToActOnBehalfOfOtherIdentity` (RBCD), `member` / `primaryGroupID` (group membership).
- **Extended and read rights**: `User-Force-Change-Password` (reset without the old one), `DS-Replication-Get-Changes` + `...-All` (the DCSync right, on the domain object), and the read rights over `msDS-ManagedPassword` / `ms-Mcs-AdmPwd` that expose gMSA and LAPS secrets.
- **`WriteGPLink`** on an OU: link a GPO to it (covered under [Group Policy](../group-policy/index.md)).

The [DACL overview](index.md) maps each of these edges to the page that abuses it.

## Enumerating ACEs

BloodHound is the practical tool because it resolves *transitive* ACL paths (A can write B, B is admin of C) rather than one object at a time; collect with ACLs enabled (see [BloodHound](../reconnaissance/bloodhound.md)) and run the dangerous-rights queries. For targeted reads:

```
# PowerView: who has rights over a specific object, resolved to names
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs |
  ? { $_.ActiveDirectoryRights -match 'GenericAll|WriteDacl|WriteOwner|GenericWrite' }

# Rights a principal you control holds across the domain
Get-DomainObjectAcl -ResolveGUIDs |
  ? { $_.SecurityIdentifier -eq '<your-SID>' }
```

```bash
# From Linux
nxc ldap <dc> -u user -p pass -M daclread -o TARGET=krbtgt
dacledit.py -action read -target 'Domain Admins' example.local/user:pass
bloodyAD --host <dc> -d example.local -u user -p pass get writable      # objects you can write
```

`-ResolveGUIDs` (and the equivalent) is important: extended rights and property writes are identified by schema GUIDs, so without resolution you cannot tell `DS-Replication-Get-Changes` (DCSync) from a harmless property write.

## Exploitation notes

- Enumerate from the perspective of **every** principal you control, including groups you are a member of and machine accounts you own; the abusable ACE is often on a group, not your user directly.
- The DACL read is the enumeration; turning each ACE into control (reset, add-member, shadow credentials, DCSync) is covered by the technique pages in this topic, mapped from the [DACL overview](index.md).
- Pay special attention to ACEs on `AdminSDHolder`, the domain root, the `krbtgt` account, and Domain/Enterprise Admins, where a single writable ACE is domain-critical.

## Tools

- **BloodHound**: transitive ACL path analysis.
- **PowerView `Get-DomainObjectAcl`**: on-host DACL reads with GUID resolution.
- **dacledit.py** (Impacket) / **NetExec daclread**: DACL reads from Linux.

## References

- The Hacker Recipes: DACL abuse
- Microsoft: Active Directory access rights and ACEs
