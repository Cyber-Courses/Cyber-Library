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

If the base is assembled as `ou=$dept,dc=example,dc=com`, controlling `$dept` moves the starting point. Replacing it with a higher node widens coverage to the whole directory:

```
dept input: anything,dc=example
base: ou=anything,dc=example,dc=com   (or, by injecting the parent)
```

The more impactful case is supplying a base that resolves higher up, such as the root `dc=example,dc=com`, combined with a subtree scope, so a query intended to list one department's users instead enumerates every entry under the domain. Where the scope itself is parameterized, switching a `base`/`one-level` search to `subtree` similarly expands what a single query returns.

Search-base injection does not change the filter logic (that is filter injection) but changes the *region* the filter runs over, so a benign filter suddenly matches across the entire directory. It leads to horizontal information disclosure, reading entries in partitions the application was scoped away from. The defense, worth noting only to explain the bug, is that the base DN and scope should be fixed server-side and never taken from the client; where they are not, this is the primitive.

## References

- RFC 4511: LDAP search operation (baseObject, scope)
- OWASP Testing Guide: Testing for LDAP Injection
