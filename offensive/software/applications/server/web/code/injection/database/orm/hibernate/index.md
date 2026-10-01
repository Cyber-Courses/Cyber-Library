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

Hibernate supports bound parameters, but HQL/JPQL assembled by string concatenation, native-SQL queries (`createNativeQuery`), and `Restrictions.sqlRestriction` or dynamic property names in the Criteria API all reintroduce injection against the underlying JDBC backend.
