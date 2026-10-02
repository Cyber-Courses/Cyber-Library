---
title: "User and group enumeration: building the principal list"
description: "Enumerating Active Directory users and groups, from authenticated LDAP reads to unauthenticated RID cycling and username validation over Kerberos, to build the target list for roasting and spraying."
keywords:
  - user enumeration
  - RID cycling
  - kerbrute
  - group membership
  - enum4linux
---

# User and group enumeration

The user and group list is the target set for nearly everything later: who to spray, who to roast, who is privileged, and which accounts are stale. How you build it depends on what access you have, from a full authenticated read to unauthenticated username guessing.

## Authenticated enumeration

With any domain credentials, read users and groups over LDAP or the SAMR interface:

```bash
nxc smb <dc> -u user -p pass --users          # SAMR user list with lastLogon, badpwd count
nxc ldap <dc> -u user -p pass --groups
net rpc group members "Domain Admins" -U 'EXAMPLE\user%pass' -S <dc>
```

From a domain host, PowerView and the built-ins give the same:

```
Get-DomainUser -Properties samaccountname,description,lastlogon
Get-DomainGroupMember "Domain Admins" -Recurse
net group "Domain Admins" /domain
```

Pull `description`, `lastLogon`, `pwdLastSet`, and `adminCount` alongside names: descriptions leak passwords, stale accounts are weak-password candidates, and `adminCount=1` flags current or former privileged accounts (the AdminSDHolder mechanism stamps protected principals this way).

## Unauthenticated discovery

Without credentials, there are still ways to build a user list:

- **RID cycling**: where a null or guest SMB session is allowed, walk the RID space to resolve SIDs to names:

  ```bash
  nxc smb <dc> -u '' -p '' --rid-brute
  enum4linux-ng -R <dc-ip>
  lookupsid.py anonymous@<dc-ip>
  ```

- **Kerberos username validation**: the AS-REQ pre-authentication response differs for valid versus invalid usernames, so a wordlist can be validated without a single login attempt (no lockout risk):

  ```bash
  kerbrute userenum -d example.local --dc <dc-ip> users.txt
  ```

- **External sources**: OSINT (company email format, LinkedIn) seeds the wordlist; harvested names plus the domain's email format produce candidate `sAMAccountName`s.

## Exploitation notes

- `kerbrute userenum` is the safest bulk validator because it never submits a password, so it does not increment the bad-password count or trigger lockout.
- Resolve group membership **recursively**: privileged access is often inherited through nested groups rather than direct membership.
- Feed the validated user list to [SPN discovery](../kerberos/spn-discovery.md) and to password spraying, and the privileged accounts to [BloodHound](../../reconnaissance/bloodhound.md) as high-value targets.

## Tools

- **NetExec (nxc)**: SAMR/LDAP user and group reads, RID brute.
- **kerbrute**: lockout-safe username validation over Kerberos.
- **enum4linux-ng / lookupsid.py**: null-session RID cycling.
- **PowerView**: authenticated user, group, and membership queries.

## References

- The Hacker Recipes: user enumeration
- Microsoft: SAMR and LDAP object model
