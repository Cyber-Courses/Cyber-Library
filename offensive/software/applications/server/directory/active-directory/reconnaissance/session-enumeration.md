---
title: "Session enumeration: finding where privileged users are logged on"
description: "Enumerating logged-on sessions and local administrators across an Active Directory domain to find where a target's credentials are available for theft, the data that turns a graph edge into a lateral-movement plan."
keywords:
  - session enumeration
  - NetSessionEnum
  - NetWkstaUserEnum
  - logged on users
  - local admin
---

# Session enumeration

Knowing *who is logged on where* is what makes targeted lateral movement possible: if a Domain Admin has an active session on a host you can reach, their credentials (or a usable Kerberos ticket) are in that host's memory, so the host becomes the stepping stone. Session enumeration is also the noisiest and most access-dependent recon, because it queries many hosts rather than just the DC.

## The data sources

Two RPC calls provide the bulk of it:

- **`NetSessionEnum`** lists sessions to a host (which users have a connection, typically to file shares), historically queryable remotely by any authenticated user, and on many hosts still is. It reveals where users are connecting from.
- **`NetWkstaUserEnum`** lists users logged on interactively to a host, but usually requires local admin on the target.
- **Local group membership** (`NetLocalGroupGetMembers`) reveals who is local admin on each host, the other half of the lateral map.

```bash
# NetExec gathers sessions, logged-on users, and local admins across a range
nxc smb <range> -u user -p pass --sessions
nxc smb <range> -u user -p pass --loggedon-users
nxc smb <range> -u user -p pass --local-auth --groups Administrators
```

From a domain host, PowerView wraps the same calls:

```
Get-NetSession -ComputerName <host>
Get-NetLoggedOn -ComputerName <host>
Find-DomainUserLocation -UserGroupIdentity 'Domain Admins'   # hunt a group's sessions
```

## BloodHound sessions

BloodHound's session and local-admin collection is exactly this data at scale: `HasSession` edges come from `NetSessionEnum`, and `AdminTo` edges from local-group membership. Collecting it (see [BloodHound](bloodhound.md)) lets the graph compute "owned principal to Domain Admin via a session on host X" automatically.

## Exploitation notes

- Session data is **volatile**: a user's session ends when they log off, so recollect immediately before acting rather than trusting an old snapshot.
- Microsoft hardened remote `NetSessionEnum` permissions in later Windows builds, so a modern, patched domain may return little without privileged access; where it still works, it is high value.
- The target of session hunting is almost always the hosts where privileged accounts (Domain Admins, service accounts, helpdesk with broad rights) are logged on, because those are where their credentials can be dumped.

## Tools

- **NetExec (nxc)**: `--sessions`, `--loggedon-users`, local-admin enumeration across ranges.
- **PowerView**: `Get-NetSession`, `Find-DomainUserLocation`.
- **BloodHound**: session and local-admin edges at scale.

## References

- [NetExec: sessions and logged-on users](https://github.com/Pennyw0rth/NetExec)
- [Impacket (fortra): netview and NetSessionEnum](https://github.com/fortra/impacket)
- [SpecterOps: BloodHound session collection](https://bloodhound.specterops.io/)
- [The Hacker Recipes: Active Directory recon](https://www.thehacker.recipes/ad/recon/)
