---
title: "DACL: turning object permissions into control"
description: "Active Directory DACL attacks: the complete map of abusable access-control entries (GenericAll, GenericWrite, WriteDacl, WriteOwner, WriteSPN, AddKeyCredentialLink, ForceChangePassword, AddMember, replication and managed-password rights) and the Linux and Windows tooling that turns each into control of a privileged principal."
keywords:
  - DACL
  - ACL abuse
  - GenericAll
  - WriteDacl
  - bloodyAD
---

# DACL

Every Active Directory object carries a discretionary access control list (DACL): a list of access-control entries (ACEs) saying which principals may read or modify it. Administrators delegate these rights liberally and almost never audit them, so AD is full of ACEs that let an ordinary principal modify a privileged one. DACL work is turning a **write** over an object into **control** of it, and BloodHound exists largely to find these edges and chain them (A can write B, B is admin of C) into a path to Domain Admin.

## The complete edge map

Each abusable right, the attribute or control-access right behind it, and where the technique is documented:

| Right (BloodHound edge) | Backed by | Turns into |
| --- | --- | --- |
| `GenericAll` | full control | anything below |
| `GenericWrite` / `WriteProperty` | property writes | SPN, key credential, delegation, membership |
| `WriteDacl` | `WRITE_DAC` | [grant yourself any right](ownership-and-acl-rewrite.md) |
| `WriteOwner` / `Owns` | `WRITE_OWNER` | [take ownership, then rewrite the DACL](ownership-and-acl-rewrite.md) |
| `ForceChangePassword` | `User-Force-Change-Password` | [reset the password](password-reset.md) |
| `AddMember` / `AddSelf` | write `member` | [join a privileged group](group-membership.md) |
| `WriteSPN` | write `servicePrincipalName` | [targeted Kerberoasting](targeted-kerberoasting.md) |
| `AddKeyCredentialLink` | write `msDS-KeyCredentialLink` | [shadow credentials](../authentication/kerberos/shadow-credentials.md) |
| `AddAllowedToAct` / `WriteAccountRestrictions` | write `msDS-AllowedToActOnBehalfOfOtherIdentity` | [resource-based delegation](../authentication/kerberos/delegation/resource-based-constrained.md) |
| `DCSync` | `DS-Replication-Get-Changes` + `-All` on the domain | [DCSync](../authentication/credentials/ntds-and-dcsync.md) |
| `ReadGMSAPassword` / `ReadLAPSPassword` | read `msDS-ManagedPassword` / `ms-Mcs-AdmPwd` | [recover managed secrets](../authentication/credentials/index.md) |
| `WriteGPLink` | write `gPLink` on an OU | [link a GPO to the OU](../group-policy/index.md) |

The right only matters for what it lets you do, so the table is the fast path: find the edge in BloodHound, jump to the technique.

## Choosing the quietest conversion

Several edges reach the same goal with very different footprints, which is the practical decision once you hold a write:

- **Shadow credentials** (where PKINIT is available) and **targeted Kerberoasting** are **non-destructive**: they do not change the victim's password or lock anyone out, so prefer them over a password reset when the account is in use.
- A **password reset** is loud and disruptive (the legitimate user loses access), so keep it for computer or stale accounts, or when nothing quieter is available.
- **RBCD** and **adding replication rights** leave a durable configuration change; **adding yourself to a group** is trivially visible in membership. Weigh persistence value against detectability.

## Pages

- **[ACL enumeration](acl-enumeration.md)**: finding the abusable ACEs with BloodHound and from Linux.
- **[Password reset](password-reset.md)**: `ForceChangePassword` / `GenericAll` to take an account over.
- **[Group membership](group-membership.md)**: `AddMember` / `AddSelf` to join a privileged group.
- **[Targeted Kerberoasting](targeted-kerberoasting.md)**: `WriteSPN` to make a target roastable.
- **[Ownership and ACL rewrite](ownership-and-acl-rewrite.md)**: `WriteOwner` / `WriteDacl`, and the Owner Rights limits that now constrain them.
- **[AdminSDHolder](adminsdholder.md)**: DACL persistence through SDProp.

## Tools

- **BloodHound**: transitive ACL path analysis and edge identification.
- **Impacket** (`dacledit.py`, `owneredit.py`): read/write ACEs and change ownership from Linux.
- **bloodyAD**: set owner, grant rights, reset passwords, add key credentials and group members from Linux.
- **PowerView** (`Add-DomainObjectAcl`, `Set-DomainObjectOwner`): on-host ACE and ownership edits.

## References

- SpecterOps: An ACE Up the Sleeve, and the BloodHound edge reference
- The Hacker Recipes: DACL abuse
