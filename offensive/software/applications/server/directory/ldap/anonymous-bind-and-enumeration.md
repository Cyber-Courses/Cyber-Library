---
title: "Anonymous bind and enumeration: reading a directory without credentials"
description: "Enumerating LDAP directories through anonymous and null binds: reading the rootDSE, naming contexts, and object data without authentication on AD, OpenLDAP, and other directories."
keywords:
  - anonymous bind
  - null bind
  - rootDSE
  - LDAP enumeration
  - unauthenticated
---

# Anonymous bind and enumeration

An LDAP **anonymous bind** authenticates with no credentials, and many directories allow it for at least part of the tree. Where it is permitted, an unauthenticated attacker reads directory structure and often object data, which is a strong unauthenticated foothold.

## Testing for anonymous access

Bind with no credentials and try to read the base:

```bash
# Anonymous bind, read the rootDSE (works on almost all directories)
ldapsearch -x -H ldap://<target> -s base -b "" "(objectClass=*)"

# Anonymous read of actual object data (works where the directory allows it)
ldapsearch -x -H ldap://<target> -b "DC=example,DC=local" "(objectClass=*)"
ldapsearch -x -H ldap://<target> -b "dc=example,dc=com" "(objectClass=person)" cn mail
```

Three outcomes:

- The bind succeeds **and** returns object data: fully anonymous-readable directory.
- The bind succeeds but object reads return nothing or an "operations error": anonymous bind allowed, data reads require auth (common on modern AD, where the rootDSE is still readable).
- The bind is rejected: anonymous binds disabled.

A true **null bind** (a bind with a username but an empty password) is a related misconfiguration that some directories accept as anonymous; test it where a simple bind with an empty password is offered.

## Per-product notes

- **Active Directory**: the rootDSE (naming contexts, supported controls) is almost always anonymously readable, which is enough for [host and domain discovery](../active-directory/authentication/reconnaissance/host-and-domain-discovery.md); object data is anonymous only if `dsHeuristics` has been loosened or the pre-Windows-2000 compatibility group includes Anonymous.
- **OpenLDAP / 389 DS**: anonymous read of the whole tree is a frequent default or misconfiguration; `userPassword` hashes may even be returned anonymously on a poorly ACL'd server.
- **Appliances** (storage, network gear, apps with embedded LDAP) often ship with permissive anonymous access.

## Exploitation notes

- Always read the rootDSE first; even when data reads are blocked it gives naming contexts, supported SASL mechanisms, and server identity.
- On OpenLDAP, request `userPassword` explicitly; where the ACL is wrong, password hashes come back anonymously and crack offline.
- Anonymous reads feed everything else: the user list for spraying, emails for phishing, and the directory structure for targeting (see [credentials in attributes](credentials-in-attributes.md)).

## Tools

- **ldapsearch**: anonymous and null bind reads.
- **NetExec (nxc) ldap**: anonymous enumeration helpers.
- **windapsearch / ldapdomaindump**: structured anonymous/authenticated dumps where allowed.

## References

- The Hacker Recipes: LDAP enumeration
- RFC 4513: LDAP authentication methods (anonymous and unauthenticated bind)
