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

## Tools

- **sqlmap**: automating exploitation of the createNativeQuery raw-SQL sink.
- **Burp Repeater and Intruder**: delivering HQL, native-SQL, and Criteria payloads.

## References

- Hibernate ORM User Guide: HQL and native SQL queries
- OWASP: SQL Injection
