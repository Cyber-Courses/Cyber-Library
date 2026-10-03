---
title: "Hibernate injection"
description: "Injection through concatenated HQL/JPQL, native SQL queries, and the Criteria API's raw fragments."
keywords:
  - Hibernate
  - HQL
  - JPQL
  - native query
  - Criteria API
---

# Hibernate

Hibernate supports bound parameters, but HQL/JPQL assembled by string concatenation, native-SQL queries (`createNativeQuery`), and raw `Restrictions.sqlRestriction` fragments reintroduce SQL injection against the underlying JDBC backend. Attacker-controlled Criteria property or sort names are a related but distinct issue: Hibernate resolves them against mapped properties rather than splicing them into SQL, so they can expose unintended fields or ordering without being SQL injection.

## Pages

- **[Criteria API injection](criteria-api-injection.md)**: The Criteria API is parameter-safe for values, but Restrictions.sqlRestriction() embeds raw SQL and user-controlled property names reach unintended columns.
- **[HQL injection](hql-injection.md)**: Concatenating user input into createQuery() HQL/JPQL bypasses Hibernate's parameter binding, exposing entity data through boolean logic and correlated subque...
- **[Native SQL injection](native-sql-injection.md)**: Hibernate's native-query APIs run raw backend SQL; concatenating user input into them yields full SQL injection including stacked queries and DBMS-specific f...

## Tools

- **sqlmap**: automating exploitation of the createNativeQuery raw-SQL sink.
- **Burp Repeater and Intruder**: delivering HQL, native-SQL, and Criteria payloads.

## References

- Hibernate ORM User Guide: HQL and native SQL queries
- OWASP: SQL Injection
