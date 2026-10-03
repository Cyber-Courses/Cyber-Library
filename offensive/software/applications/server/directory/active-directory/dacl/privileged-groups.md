---
title: "Privileged groups: escalating through built-in group membership"
description: "What membership of Active Directory's built-in privileged groups grants an attacker: DnsAdmins, Backup Operators, Server Operators, Print Operators, Account Operators, Schema Admins, and others, and the escalation each enables to SYSTEM or Domain Admin."
keywords:
  - privileged groups
  - DnsAdmins
  - Backup Operators
  - Server Operators
  - SeLoadDriverPrivilege
---

# Privileged groups

Several built-in Active Directory groups are **Domain Admin in all but name**: their membership grants a privilege or access that converts directly to SYSTEM on a domain controller or to the domain database. Getting *into* such a group is a [DACL](index.md) problem (a weak ACL, a password reset, a [group write](group-membership.md)); this page is about what each group lets you *do* once you are a member, which is where [ACL enumeration](acl-enumeration.md) and BloodHound paths usually terminate.

## The high-value groups

- **[DnsAdmins](dnsadmins.md)**: load a DLL into the DNS service (SYSTEM on the DC) via `ServerLevelPluginDll`.
- **[Backup Operators](backup-operators.md)**: `SeBackupPrivilege` reads `NTDS.dit` and the SYSTEM hive past their ACLs, every domain hash.
- **[Server Operators](server-operators.md)**: reconfigure a service's binary path to run as LocalSystem on the DC.

## The rest, and what they grant

- **Print Operators**: holds **`SeLoadDriverPrivilege`** and can log on to DCs; load a malicious or vulnerable signed driver to reach kernel/SYSTEM code execution.
- **Account Operators**: manage most **non-protected** users and groups; use it to take over any unprotected account or add members to unprotected groups (not the [AdminSDHolder](adminsdholder.md)-protected ones). Covered under [group membership](group-membership.md).
- **Schema Admins**: modify the **schema**, including a class's `defaultSecurityDescriptor`, so every future object of that class carries an ACE you choose, a slow but forest-wide backdoor.
- **Group Policy Creator Owners**: create GPOs, which combined with a link right is code execution (see [Group Policy](../group-policy/index.md)).
- **Pre-Windows 2000 Compatible Access**: grants broad **read** over the directory to its members (historically Everyone/Anonymous), widening unauthenticated or low-priv enumeration.
- **Enterprise/Key Admins**, **DHCP Administrators**, and similar delegated groups each carry narrower but real escalations worth checking per environment.

## Exploitation notes

- These groups are the usual **end of a BloodHound path**: an ACL edge leads to membership, and membership leads to SYSTEM/DA through the technique above, so enumerate your effective group memberships, including nested and `primaryGroupID`, not just direct ones.
- Most of these groups are **protected** (SDProp/AdminSDHolder), so you rarely add yourself via a weak ACL; you reach them by compromising an existing member, except **DnsAdmins**, which is **not** protected and is frequently delegated.
- Membership that grants a **privilege** (SeBackup, SeLoadDriver) only matters where you can log on or act on the target (DC), so confirm the logon/right applies there.

## Tools

- **BloodHound**: maps edges into these groups and flags the membership.
- **PowerView / NetExec**: enumerate effective membership and the groups' rights.

## References

- [HackTricks: privileged groups and token privileges](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges.html)
- [qazeer notes: operators to Domain Admins](https://notes.qazeer.io/active-directory/exploitation-operators_to_domain_admins)
- [Microsoft: Appendix B, privileged accounts and groups in Active Directory](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory)
