---
title: "LDAP search base injection"
description: "Controlling the search base or scope an application passes to an LDAP query so it reads a wider or different partition of the directory than intended."
keywords:
  - search base injection
  - base DN
  - subtree scope
  - LDAP partition
  - directory read
---

# Search base injection

An LDAP search takes three inputs: the base DN (where to start), the scope (base, one-level, or subtree), and the filter. Applications usually fix the base and scope and vary only the filter, but where the base or scope is built from input, an attacker can redirect the search to read more of the tree.

If the base is assembled as `ou=$dept,dc=example,dc=com`, injecting RDNs into `$dept` can only move the start sideways or deeper within that fixed suffix (LDAP DNs have no parent-traversal syntax, so you cannot climb above `dc=example,dc=com` from a fragment), and extra RDNs just prepend below it:

```
dept input: staff,ou=contractors
base: ou=staff,ou=contractors,dc=example,dc=com
```

The impactful case is when the application takes the whole base DN, or the scope, from the client rather than only a fragment. A client-supplied base of the root `dc=example,dc=com` with a subtree scope turns a query meant to list one department into an enumeration of every entry under the domain. Likewise, where the scope is parameterized, switching a `base`/`one-level` search to `subtree` expands what a single query returns. So the primitive depends on how much of the base and scope the input controls: a suffixed fragment redirects within a subtree, while a full base or scope widens to the whole directory.

Search-base injection does not change the filter logic (that is filter injection) but changes the *region* the filter runs over, so a benign filter suddenly matches across the entire directory. It leads to horizontal information disclosure, reading entries in partitions the application was scoped away from. The defense, worth noting only to explain the bug, is that the base DN and scope should be fixed server-side and never taken from the client; where they are not, this is the primitive.

## Tools

- **ldapsearch**: issue searches with varied base DN and scope to confirm widening.
- **windapsearch**: enumerate the tree reachable from a redirected base.
- **Burp Repeater**: craft requests that control the base or scope parameter.

## References

- RFC 4511: LDAP search operation (baseObject, scope)
- OWASP Testing Guide: Testing for LDAP Injection
