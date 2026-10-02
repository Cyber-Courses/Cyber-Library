---
title: "Hibernate native SQL injection: createNativeQuery() / createSQLQuery() with concatenation"
description: Hibernate's native-query APIs run raw backend SQL; concatenating user input into them yields full SQL injection including stacked queries and DBMS-specific functions.
keywords:
  - Hibernate
  - native SQL injection
  - createNativeQuery()
  - createSQLQuery()
  - JDBC
  - SQL injection
---

# Hibernate native SQL injection

Hibernate lets applications drop out of HQL and run **native backend SQL** through `createNativeQuery()` (JPA) or the legacy `createSQLQuery()`. These execute the raw string against the underlying JDBC connection, so concatenating user input produces full SQL injection in the database's own dialect, more powerful than [HQL injection](hql-injection.md) because every native construct is available.

## Vulnerable pattern

```java
// id from the request
String sql = "SELECT * FROM users WHERE id = " + id;
Query q = session.createNativeQuery(sql);
```

The safe form uses placeholders (`WHERE id = ?1` / `:id` with `setParameter`); the injectable form interpolates directly.

## Exploitation

Native queries expose the real tables and the DBMS feature set. Numeric context:

```
1 OR 1=1
0 UNION SELECT username, password, NULL FROM users
```

String context:

```
' UNION SELECT username, password FROM users --
```

Depending on the backend and JDBC settings, **stacked queries** may be available (`; UPDATE ...`), and DBMS-specific primitives apply, `pg_sleep()` / `BENCHMARK()` for time-based blind, `CONVERT()`/`CAST()` for error-based extraction, and file or command primitives where the database and privileges allow. Confirm the backend first (via error strings, version functions such as `@@version` / `version()`), then select the matching dialect payloads.

When the native query maps results to an entity via `addEntity()`, the selected column order must match the entity mapping; with `createNativeQuery(sql)` returning `Object[]`, a `UNION` can project arbitrary columns for direct read.

## References

- [Hibernate ORM: Native SQL queries](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#sql)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
