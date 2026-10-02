---
title: "DACL: abusing Active Directory object permissions"
description: "Abusing discretionary access control lists on Active Directory objects: turning rights like GenericAll, GenericWrite, WriteDacl, WriteOwner, and ForceChangePassword over a principal into control of it, through password resets, group membership, targeted roasting, shadow credentials, delegation, and replication rights."
keywords:
  - DACL
  - ACL abuse
  - GenericAll
  - WriteDacl
  - bloodhound
---

# DACL

Every Active Directory object carries a discretionary access control list (DACL) that says who may read and modify it. Because administrators delegate rights liberally and rarely audit them, these ACLs are riddled with edges that let an ordinary principal modify a privileged one. DACL abuse is the art of turning a **write** over an object into **control** of it, and BloodHound exists largely to find these edges and chain them into a path to Domain Admin.

## The rights that matter

- **GenericAll / GenericWrite**: full or broad write over an object, the master keys that enable every abuse below.
- **WriteDacl / WriteOwner**: rewrite the object's ACL or take ownership, then grant yourself the rights you need.
- **ForceChangePassword**: reset a user's password without knowing the old one.
- **AddMember / Self (membership)**: add yourself (or a controlled account) to a group.
- **AllExtendedRights**: includes the replication rights that enable DCSync.

## What a write becomes

A DACL edge is only useful for what it lets you do; the common conversions route into the authentication techniques:

- **Reset the password** (ForceChangePassword) to take over the account directly.
- **Add to a privileged group** (AddMember) to inherit its rights.
- **Write an SPN** on a user to make it roastable, then crack it ([roasting](../authentication/kerberos/roasting.md)).
- **Write `msDS-KeyCredentialLink`** to authenticate as the object ([shadow credentials](../authentication/kerberos/shadow-credentials.md)).
- **Write the delegation attribute** to impersonate to it ([resource-based delegation](../authentication/kerberos/delegation/resource-based-constrained.md)).
- **Grant replication rights** to a principal to enable [DCSync](../authentication/credentials/ntds-and-dcsync.md).

## Pages

- **[ACL enumeration](acl-enumeration.md)**: finding the abusable rights with BloodHound and LDAP.

## References

- The Hacker Recipes: DACL abuse
- SpecterOps: BloodHound and AD ACL attack paths
