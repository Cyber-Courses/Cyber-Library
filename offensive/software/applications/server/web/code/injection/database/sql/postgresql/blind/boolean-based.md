---
title: "Boolean character extraction in PostgreSQL blind injection"
description: "The substring and ascii character oracle for PostgreSQL blind SQL injection, extracting values with a binary search over code points."
keywords:
  - substring
  - ascii
  - binary search
  - blind extraction
  - character oracle
---

# Boolean extraction

The character oracle reads a value one position at a time. `substring(value FROM pos FOR 1)` (or `substring(value, pos, 1)`) returns the character at `pos`, and `ascii(...)` gives its code point for a comparison the response reveals as true or false.

Extract a value with a binary search over each position. For the first character of the first role's password hash:

```sql
' AND ascii(substring((SELECT passwd FROM pg_shadow LIMIT 1),1,1))>64-- 
' AND ascii(substring((SELECT passwd FROM pg_shadow LIMIT 1),1,1))>96-- 
' AND ascii(substring((SELECT passwd FROM pg_shadow LIMIT 1),1,1))=116-- 
```

Each `>` test halves the candidate range, so a printable character resolves in about seven requests. Advance the `substring` position to walk the string and stop at the length found earlier.

When comparison operators are filtered, `substring(...) = 't'` tests equality directly, and the SQL-standard `substring(value FROM pos FOR 1)` form avoids the comma that some filters block. Against an unprivileged role that cannot read `pg_shadow`, point the subquery at an application table instead. Like all blind extraction, this is heavily automated, but the per-request comparison is the reliable primitive underneath the tooling.

## References

- PostgreSQL Documentation: `substring`, `ascii`
- PortSwigger Web Security Academy: Blind SQL injection
