---
title: "Authenticated LDAP enumeration: pulling the full object set"
description: "Enumerating an LDAP directory with any valid bind: dumping users, groups, and structure, paging large result sets, and querying the Global Catalog for a forest-wide view."
keywords:
  - authenticated enumeration
  - LDAP bind
  - paged results
  - global catalog
  - ldapsearch
---

# Authenticated enumeration

Any valid credential, however low-privileged, usually unlocks a near-complete read of a directory. On AD a single domain user can read almost every object; on OpenLDAP a bound user reads whatever the ACLs grant, which is often broad. Authenticated enumeration is where the full inventory is built.

## Binding and dumping

```bash
# Simple bind (cleartext over LDAP; prefer LDAPS where available)
ldapsearch -x -H ldap://<target> -D 'cn=user,dc=example,dc=com' -w 'password' \
  -b 'dc=example,dc=com' "(objectClass=*)"

# AD-style bind
ldapsearch -x -H ldap://<dc> -D 'EXAMPLE\user' -w 'password' \
  -b 'DC=example,DC=local' "(objectClass=user)" sAMAccountName
```

For AD, the object-model-aware queries (UAC bit filters, SPNs, delegation) are covered under [LDAP enumeration](../active-directory/authentication/reconnaissance/ldap-enumeration.md); the mechanics below apply to any directory.

## Paging large results

Directories cap how many entries a single search returns (AD's `MaxPageSize` is 1000 by default), so a naive dump silently truncates. Use the simple paged-results control:

```bash
# ldapsearch handles paging with -E pr=
ldapsearch ... -E pr=1000/noprompt "(objectClass=user)" sAMAccountName

# ldapdomaindump and nxc page automatically
ldapdomaindump -u 'EXAMPLE\user' -p 'password' <dc>
```

If a dump returns exactly 1000 (or another round number) of entries, assume truncation and switch to a paged query.

## Global Catalog

AD's **Global Catalog** (ports 3268 plaintext, 3269 TLS) holds a partial, forest-wide replica, so a single query against it enumerates every domain in the forest at once:

```bash
ldapsearch -x -H ldap://<gc>:3268 -D 'EXAMPLE\user' -w 'password' \
  -b '' -s sub "(objectClass=user)" sAMAccountName
```

## Exploitation notes

- Bind over **LDAPS** (636) or use StartTLS where possible; a simple bind over plain LDAP sends the password in cleartext and is itself capturable on the wire.
- Always page; truncated enumeration quietly hides the exact high-value accounts you are looking for.
- Request the attributes that matter (`description`, `memberOf`, UAC, `servicePrincipalName`) rather than whole objects, both for speed and to surface the [credential-bearing fields](credentials-in-attributes.md) directly.

## Tools

- **ldapsearch** with `-E pr=`: paged authenticated reads.
- **ldapdomaindump**: full offline directory dump with paging.
- **NetExec (nxc) ldap**: targeted authenticated extractions.

## References

- The Hacker Recipes: LDAP enumeration
- Microsoft: LDAP paging and MaxPageSize
