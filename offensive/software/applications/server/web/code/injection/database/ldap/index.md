---
title: "LDAP injection: filter, DN, and search-base abuse"
description: "Exploiting LDAP injection when application code builds directory filters, distinguished names, or search bases from untrusted input, including authentication bypass and blind extraction."
keywords:
  - LDAP injection
  - LDAP filter injection
  - DN injection
  - search base injection
  - directory query
---

# LDAP

LDAP injection happens when an application builds a directory operation from untrusted input without escaping it, the directory equivalent of SQL injection. It appears wherever a filter, a distinguished name (DN), or a search base is assembled by string concatenation.

The core syntax is the search filter, written in prefix notation: `(attr=value)`, combined with `&` (AND), `|` (OR), and `!` (NOT), as in `(&(uid=jane)(objectClass=person))`. The wildcard `*` matches any value. RFC 4515 requires five characters to be escaped inside a filter value (`*` `(` `)` `\` and NUL, as `\2a \28 \29 \5c \00`); when the application does not escape them, injected parentheses and wildcards change the filter's logic. DNs have their own escaping rules (RFC 4514) for characters such as `,` `+` `=` and `"`.

Unlike SQL, LDAP has no `UNION` or comments, and errors are usually terse, so extraction leans on wildcards and boolean response differences rather than rich in-band output. The impact ranges from authentication bypass and authorization flaws to disclosing directory attributes.

## Techniques

- **[Filter injection](filter-injection.md)**: inject parentheses, operators, and wildcards into a search filter.
- **[Authentication bypass](authentication-bypass.md)**: subvert an LDAP-backed login.
- **[Blind](blind.md)**: infer attribute values from wildcard and boolean responses.
- **[DN injection](dn-injection.md)**: alter the distinguished name used for bind or modify.
- **[Search base injection](search-base-injection.md)**: widen or move the search base to read more of the tree.

## References

- RFC 4515 (LDAP search filters) and RFC 4514 (distinguished names)
- OWASP Testing Guide: Testing for LDAP Injection
