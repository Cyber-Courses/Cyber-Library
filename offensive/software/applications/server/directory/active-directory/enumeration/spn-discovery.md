---
title: "SPN discovery: locating service accounts for roasting"
description: "Finding service principal names in Active Directory to identify kerberoastable accounts: querying servicePrincipalName over LDAP, setspn, and the overlap with high-value service identities."
keywords:
  - SPN
  - servicePrincipalName
  - kerberoasting target
  - setspn
  - service account
---

# SPN discovery

A **service principal name** (SPN) maps a service instance to the account that runs it. Any account that has an SPN can have a service ticket requested for it by any domain user, and that ticket is encrypted with the account's password hash, which is the precondition for Kerberoasting. Discovering which accounts carry SPNs tells you exactly which identities are roastable, and the most valuable ones are usually service accounts with high privilege and old, human-chosen passwords.

## Finding accounts with SPNs

The query is a single LDAP filter: user objects that have a `servicePrincipalName` set.

```bash
# LDAP filter for kerberoastable users (exclude the krbtgt account)
ldapsearch ... '(&(objectClass=user)(servicePrincipalName=*)(!(sAMAccountName=krbtgt)))' \
  sAMAccountName servicePrincipalName memberOf

# Tooling that lists SPNs and their accounts
nxc ldap <dc> -u user -p pass --kerberoasting out.txt
GetUserSPNs.py example.local/user:pass -dc-ip <dc>      # add -request to also roast
setspn -T example.local -Q */*                          # from a domain-joined host
```

From a domain host, PowerView:

```
Get-DomainUser -SPN -Properties samaccountname,serviceprincipalname,memberof,pwdlastset
```

## Reading the results

Prioritize by two signals:

- **Privilege**: cross-reference the account's `memberOf` against privileged groups. A service account that is a member of Domain Admins (common for legacy SQL, backup, or monitoring services) makes a roast immediately domain-critical.
- **Password age**: an old `pwdLastSet` on a human-managed service account suggests a static, crackable password. Machine accounts and gMSAs have long random passwords and are not worth roasting.

Note the difference between user SPNs (roastable) and the SPNs on computer accounts: computer-account hashes are effectively uncrackable, so Kerberoasting targets *user* accounts with SPNs.

## Exploitation notes

- Finding the SPN is enumeration; requesting and cracking the ticket is covered under Kerberos roasting in the authentication and credentials section.
- Where a principal you control has `GenericAll`/`GenericWrite` over a target user, you can *write* an SPN onto it and roast it on demand (targeted Kerberoasting), so SPN discovery also informs which write primitives are worth using.
- A single low-privileged domain user is enough to enumerate every SPN in the domain.

## Tools

- **GetUserSPNs.py** (Impacket): list and request service tickets.
- **NetExec (nxc) ldap --kerberoasting**: enumerate and extract.
- **PowerView `Get-DomainUser -SPN`**: on-host discovery.
- **setspn**: native Windows SPN query.

## References

- The Hacker Recipes: Kerberoast
- Microsoft: Service principal names
