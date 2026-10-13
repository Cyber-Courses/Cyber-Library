---
title: "Database injection"
order: 3
description: "Untrusted input altering the query a server sends to its data store, SQL, NoSQL, ORM escape hatches, and directory services."
keywords:
  - database injection
  - SQL injection
  - NoSQL injection
  - ORM injection
  - LDAP injection
---

# Database

Database injection is where untrusted input alters the query a server sends to its data store. It spans classic **SQL** (string-built queries across engines), **NoSQL** (operator and syntax injection in document, key-value, wide-column, and graph stores), **ORM** escape hatches that reintroduce raw queries, and directory services such as **LDAP**. Impact ranges from authentication bypass and data disclosure to, depending on the engine and privileges, file access and command execution.

## Subtopics

- **[LDAP](ldap/index.md)**: Exploiting LDAP injection when application code builds directory filters, distinguished names, or search bases from untrusted input, including authentication...
- **[NoSQL](nosql/index.md)**: Injection against non-relational data stores, where attacker-controlled objects, operators, or query-language fragments are interpreted by the driver rather...
- **[ORM](orm/index.md)**: Object-relational mappers parameterize normal queries, but every one exposes escape hatches that reopen injection, organized by framework.
- **[SQL](sql/index.md)**: SQL injection organized by database engine, because the exploitable syntax, functions, catalog, and file and command primitives differ sharply between MySQL,...
