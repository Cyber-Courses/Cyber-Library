---
title: "LDAP filter injection"
description: "Injecting parentheses, boolean operators, and wildcards into an unescaped LDAP search filter to change its logic and read unintended entries."
keywords:
  - LDAP filter injection
  - wildcard
  - boolean operator
  - RFC 4515
  - filter escaping
---

# Filter injection

When user input is concatenated into an LDAP search filter without escaping, the attacker controls the filter's structure, not just a value. The injectable characters are the five RFC 4515 metacharacters, above all the parentheses and the `*` wildcard.

Consider a people-search filter `(&(objectClass=person)(cn=$input))`. A wildcard returns everything:

```
input: *
filter: (&(objectClass=person)(cn=*))
```

Injecting parentheses and an operator rewrites the logic. Closing the current clause and appending a new one lets the attacker add conditions the developer never wrote:

```
input: *)(uid=admin)
filter: (&(objectClass=person)(cn=*)(uid=admin))
```

Boolean operators broaden a match. Supplying input that opens an `|` (OR) group makes the filter true for far more entries, and metadata can be disclosed by testing for attributes:

```
input: *)(|(objectClass=*
filter: (&(objectClass=person)(cn=*)(|(objectClass=*))
```

Exactly how an injected extra filter after the outer `)` is handled depends on the client library and server: some use only the first complete filter, which still matches broadly, while others reject a malformed string. The reliable primitive is the wildcard plus injected clauses inside the existing group. Because the filter is prefix-structured, the attacker works by closing groups with `)` and opening new ones with `(`, mirroring how SQL injection closes quotes, and the same unescaped input feeds the authentication-bypass and blind techniques.

## Tools

- **ldapsearch**: test injected filter strings directly against the directory.
- **windapsearch**: enumerate directory objects and attributes worth targeting.
- **Burp Repeater**: craft operator-payload requests and compare responses.
- **python-ldap**: script filter payloads programmatically.

## References

- RFC 4515: LDAP search filter string representation
- OWASP Testing Guide: Testing for LDAP Injection
