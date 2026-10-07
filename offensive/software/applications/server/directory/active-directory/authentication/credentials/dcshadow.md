---
title: "DCShadow: pushing changes through a rogue domain controller"
order: 9
description: "Registering a rogue domain controller to inject arbitrary changes into Active Directory over the replication protocol (DCShadow), writing SID history, group membership, or ACLs directly into the directory while bypassing most logging."
keywords:
  - DCShadow
  - rogue domain controller
  - DRSUAPI
  - IDL_DRSReplicaAdd
  - domain persistence
---

# DCShadow

[DCSync](ntds-and-dcsync.md) abuses replication to **read** the directory; **DCShadow** abuses it to **write**. You temporarily register a **rogue domain controller** in the configuration, use it to push arbitrary changes (a SID history value, a group membership, an ACL) into a real DC over the replication protocol, then deregister it. Because the change arrives as normal DC-to-DC replication rather than an ordinary LDAP write, it bypasses most directory-change logging and SIEM correlation. It is a domain-dominance and persistence primitive, not a way in: you already need the rights to replicate.

## How it works

DCShadow (Mimikatz `lsadump::dcshadow`) runs in two parts:

1. **Register and serve** (as **SYSTEM** on any domain-joined host): Mimikatz adds the SPNs and `nTDSDSA`/server objects that make the host look like a DC, then hosts the minimal **MS-DRSR** RPC server that will serve the forged data.
2. **Push** (as a user with replication rights, typically **Domain/Enterprise Admin**): a second Mimikatz session triggers `IDL_DRSReplicaAdd`, forcing a legitimate DC to pull the staged changes from the rogue one.

```text
# Session 1 (SYSTEM): stage the change and host the rogue DC
lsadump::dcshadow /object:targetUser /attribute:primaryGroupID /value:512
# Session 2 (Domain Admin): force replication to push it
lsadump::dcshadow /push
```

## What you push

Anything in the directory, chosen to be quiet and durable:

- **`sIDHistory`** on an account you control, injecting a privileged SID (domain admins, enterprise admins) for standing access.
- **`primaryGroupID` = 512** to make a user a Domain Admin without appearing in the group's `member` list.
- **`ntSecurityDescriptor`** edits (for example on [AdminSDHolder](../../dacl/adminsdholder.md)) to plant a durable ACL backdoor.
- Attribute changes that undo defenders' cleanups, re-applied on demand.

## Exploitation notes

- The rogue DC exists only for the push and is torn down after, so there is no lasting rogue-DC object; what persists is the **change** you injected.
- The replication path means the change does not generate the directory-service modification events an LDAP write would, which is the whole point.
- It needs replication rights (DA/EA, or a principal granted `DS-Replication-Get-Changes` plus the `DS-Install-Replica`/write rights on the configuration), so it is post-compromise; pair it with a [DCSync](ntds-and-dcsync.md) read to know exactly what to overwrite.
- `minimal`/`DcShadow` variants reduce the rights needed in some setups, but the DA-level case is the reliable one.

## Tools

- **Mimikatz** (`lsadump::dcshadow`, `lsadump::dcshadow /push`): the reference implementation (two sessions).
- **Impacket** (`ntlmrelayx`/`secretsdump` for the companion read): reconnaissance of what to overwrite.

## References

- [DCShadow (Benjamin Delpy & Vincent Le Toux)](https://www.dcshadow.com/)
- [MITRE ATT&CK T1207: Rogue Domain Controller](https://attack.mitre.org/techniques/T1207/)
- [Hacking Articles: domain persistence, DCShadow](https://www.hackingarticles.in/domain-persistence-dc-shadow-attack/)
