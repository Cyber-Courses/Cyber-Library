---
title: "Database injection: SQL, NoSQL, ORM misuse, and second-order query flaws"
description: Server-side code that composes query or directory filter languages from untrusted input, including SQL, NoSQL, and LDAP, without a safe query API.
keywords:
  - SQL injection
  - NoSQL injection
  - ORM
  - LDAP injection
---

# Database injection

Tainted data in string-built `WHERE` clauses, `ORDER BY` injection, and NoSQL operator objects (for example, `$where`, `$regex`) are the classic database injection family. **Safe query APIs and bind parameters** choke most of this off in greenfield code; **raw** concatenation in migrations, reports, ad hoc admin tools, and `whereRaw` / string-built sort keys brings it back—those are the sinks you hunt in review and testing.

## Topics

| Area | Path |
|------|------|
| SQL | [SQL](sql/index.md) — Portable notes and per-engine hubs (MySQL, PostgreSQL, …) |
| NoSQL | [NoSQL](nosql/index.md) — Operator injection and document query shapes |
| LDAP | [LDAP](ldap/index.md) — Filter and DN assembly |
| ORM / query builders | [ORM](orm/index.md) — Raw fragments, second-order, dynamic sort columns |

## See also

- [Injection (parent)](../index.md)
