---
title: "LDAP enumeration: querying Active Directory objects directly"
description: "Reading Active Directory over LDAP: authenticated and anonymous queries, useful search filters for users, computers, groups, and service accounts, and bulk dumps of the directory."
keywords:
  - LDAP enumeration
  - ldapsearch
  - LDAP filter
  - userAccountControl
  - ldapdomaindump
---

# LDAP enumeration

AD exposes almost its entire contents over LDAP, and any authenticated domain user can read the vast majority of it. Targeted LDAP queries are how you pull specific slices (service accounts, computers with delegation, users with weak flags) without collecting the whole graph.

## Querying with filters

LDAP search filters select objects by attribute. The workhorse attributes are `objectClass`, `sAMAccountName`, and the bit-field `userAccountControl` (UAC):

```bash
# All users
ldapsearch -x -H ldap://<dc> -D 'EXAMPLE\user' -w 'Password1' \
  -b 'DC=example,DC=local' '(&(objectClass=user)(objectCategory=person))' sAMAccountName

# All computers
ldapsearch ... '(objectClass=computer)' dNSHostName operatingSystem

# Domain admins (memberOf the group)
ldapsearch ... '(memberOf=CN=Domain Admins,CN=Users,DC=example,DC=local)' sAMAccountName
```

UAC bit filters surface the highest-value targets directly:

```
# Kerberoastable: user accounts with a servicePrincipalName (objectCategory=person
# excludes computer accounts, which are a subclass of user and do not crack)
(&(objectClass=user)(objectCategory=person)(servicePrincipalName=*)(!(sAMAccountName=krbtgt)))

# AS-REP roastable: DONT_REQ_PREAUTH (0x400000)
(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))

# Unconstrained delegation: TRUSTED_FOR_DELEGATION (0x80000)
(userAccountControl:1.2.840.113556.1.4.803:=524288)

# Password never expires (0x10000) OR password not required (0x20): one bit test per flag
(|(userAccountControl:1.2.840.113556.1.4.803:=65536)(userAccountControl:1.2.840.113556.1.4.803:=32))
```

The OID `1.2.840.113556.1.4.803` is the bitwise-AND matching rule, essential for reading UAC flags.

## Bulk dumps

For a full offline copy, dump every object and attribute:

```bash
# ldapdomaindump: HTML/JSON/greppable dump of users, groups, computers, policies
ldapdomaindump -u 'EXAMPLE\user' -p 'Password1' <dc-ip>

# NetExec LDAP module: targeted extractions
nxc ldap <dc> -u user -p pass --users --groups --computers
nxc ldap <dc> -u user -p pass --kerberoasting out.txt --asreproast out.txt
```

## Anonymous and null reads

Some DCs permit anonymous LDAP binds or expose data through null SMB sessions. Where allowed, the rootDSE, schema, and sometimes object data read without credentials (see [LDAP anonymous bind](../../../ldap/anonymous-bind-and-enumeration.md)). Even when object reads require auth, the rootDSE discovery of naming contexts does not.

## Exploitation notes

- Read the interesting attributes, not just names: `description` and `info` fields frequently contain passwords; `ms-MCS-AdmPwd` (LAPS) and `msDS-ManagedPassword` (gMSA) hold machine and service credentials for principals allowed to read them (see [credentials in attributes](../../../ldap/credentials-in-attributes.md)).
- Query the Global Catalog (port 3268) for a forest-wide view across all domains in one search.
- Everything you pull here feeds [BloodHound](bloodhound.md); the targeted filters above are for when you want a specific answer fast.

## Tools

- **ldapsearch**: raw filtered queries.
- **ldapdomaindump**: full offline directory dump.
- **NetExec (nxc) ldap**: targeted users/groups/computers and roasting extraction.
- **windapsearch**: convenience queries (privileged users, computers, GPOs).

## References

- Microsoft: LDAP matching rules and userAccountControl flags
- The Hacker Recipes: LDAP enumeration
