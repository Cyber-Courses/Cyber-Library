---
title: "Database injection"
description: "Untrusted input altering the query a server sends to its data store — SQL, NoSQL, ORM escape hatches, and directory services."
keywords:
  - database injection
  - SQL injection
  - NoSQL injection
  - ORM injection
  - LDAP injection
---

# Database

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

Database injection is where untrusted input alters the query a server sends to its data store. It spans classic **SQL** (string-built queries across engines), **NoSQL** (operator and syntax injection in document, key-value, wide-column, and graph stores), **ORM** escape hatches that reintroduce raw queries, and directory services such as **LDAP**. Impact ranges from authentication bypass and data disclosure to — depending on the engine and privileges — file access and command execution.
