---
title: "LDAP DN injection"
description: "Altering the distinguished name an application builds for a bind or modify operation by injecting RDN separators and unescaped DN metacharacters."
keywords:
  - DN injection
  - distinguished name
  - RDN separator
  - RFC 4514
  - bind DN
---

# DN injection

A distinguished name uniquely identifies a directory entry, for example `uid=jane,ou=people,dc=example,dc=com`. When an application assembles a DN from input (to bind as a user, or to locate an entry to read or modify), unescaped input can change which entry the DN points to.

DNs use a different metacharacter set from filters (RFC 4514): `,` `+` `=` `"` `\` `<` `>` `;`, a leading `#`, and leading or trailing spaces must be escaped. The comma and `+` are the dangerous ones, because they separate relative distinguished names (RDNs) and combine multi-valued RDNs.

Against a template `uid=$input,ou=people,dc=example,dc=com`, injecting a comma reparents or retargets the entry:

```
input: jane,ou=admins
DN: uid=jane,ou=admins,ou=people,dc=example,dc=com
```

This shifts the DN into a different subtree, so a bind or lookup resolves to an entry the developer did not intend. A multi-valued RDN via `+` can attach an extra attribute to the name, and an injected `=` can redefine the attribute type of the RDN.

DN injection is narrower than filter injection because a DN must still resolve to a real entry, so it is used to pivot to a neighbouring or higher-privileged entry (for example moving a bind DN into an administrative OU) rather than to match broadly. Where the input feeds the bind DN of an authentication step, controlling it can select which account the application binds as.

## References

- RFC 4514: LDAP string representation of distinguished names
- OWASP Testing Guide: Testing for LDAP Injection
