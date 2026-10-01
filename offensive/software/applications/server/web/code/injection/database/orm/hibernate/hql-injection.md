---
title: "HQL injection: Hibernate Query Language built from untrusted input"
description: Concatenating user input into createQuery() HQL/JPQL bypasses Hibernate's parameter binding, exposing entity data through boolean logic and correlated subqueries over mapped entities.
keywords:
  - Hibernate
  - HQL injection
  - JPQL injection
  - createQuery()
  - parameter binding
  - SQL injection
---

# HQL injection

Hibernate Query Language (HQL), and the JPA equivalent JPQL, is an object-oriented query language that Hibernate translates to SQL. It supports named/positional **parameters**, but when application code concatenates user input into the query string passed to `createQuery()`, those protections are skipped and the query becomes injectable.

## Vulnerable pattern

```java
// username from the request
String hql = "FROM User WHERE username = '" + username + "'";
Query<User> q = session.createQuery(hql, User.class);
```

The safe form binds a parameter (`WHERE username = :u` with `setParameter("u", username)`); the injectable form interpolates the value into the string.

## Exploitation

HQL operates over **entities and their fields**, not raw tables, which shapes the payloads. Boolean breakout works as in SQL:

```
' OR '1'='1
' OR 1=1 --
```

HQL has no `UNION` (set operators arrived only in Hibernate ORM 6.1, and plain JPQL still lacks them), so retrieval pivots through **subqueries and entity navigation** instead:

```
' OR username='admin' --
' OR (SELECT u2.password FROM User u2 WHERE u2.username='admin') LIKE 'a%' --
```

Because HQL resolves field access against the mapping, a correlated subquery or an association path reads sensitive properties of related entities the query never intended to expose (e.g. `user.credentials.passwordHash`). For set-based extraction like `UNION SELECT`, which HQL cannot express, pivot to the native-query sink (see [Native SQL Injection](native-sql-injection.md)) if the application also exposes one.

Blind extraction uses the same boolean/substring inference as SQL injection, craft conditions on `SUBSTRING(u.password,1,1)='a'` and observe result differences. Hibernate underneath runs on any JDBC backend (PostgreSQL, MySQL, Oracle, SQL Server), so the generated SQL dialect follows the configured database.

## References

- [Hibernate ORM: HQL/JPQL](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#hql)
- [OWASP: Testing for ORM Injection](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
