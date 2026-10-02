---
title: "Java ORM injection"
description: "Hibernate and JPA bind parameters on their query APIs, but HQL/JPQL string building, native queries, and dynamic criteria construction from untrusted input defeat that binding."
keywords:
  - Java ORM
  - Hibernate injection
  - HQL
  - JPQL
  - native query
---

# Java

JVM persistence runs through Hibernate and the JPA it implements. Queries are safe when written with bound parameters through `setParameter`, and injectable when HQL, JPQL, or native SQL is assembled by string concatenation.

## The ORM here

- **[Hibernate](hibernate/index.md)**: HQL built with `createQuery()` concatenation, native SQL through `createNativeQuery()`, and dynamic criteria built from input.

## The shared pattern

HQL and JPQL operate over mapped entities and their fields rather than raw tables, which shapes the payloads but does not make them safe: a concatenated `createQuery()` string accepts the same boolean and subquery breakouts as SQL. Native queries drop to raw SQL and carry the full injection surface, including `UNION` and stacked statements where the driver allows them. Dynamic criteria assembled by selecting operators or fields from input is the ORM-API form of the same flaw. The fix in every case is `setParameter` binding and a fixed query shape, so the absence of those is the signal.

## Tools

- **sqlmap**: exploiting Hibernate native-query and raw-fragment sinks.
- **Burp Repeater and Intruder**: crafting HQL, native-SQL, and criteria payloads.

## References

- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Hibernate ORM: HQL/JPQL](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#hql)
