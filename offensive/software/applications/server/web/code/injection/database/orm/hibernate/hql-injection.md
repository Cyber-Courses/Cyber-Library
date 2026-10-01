---
title: "HQL injection: Hibernate Query Language built from untrusted input"
description: Concatenating user input into createQuery() HQL/JPQL bypasses Hibernate's parameter binding, exposing entity data through boolean logic and UNION-style object queries.
keywords:
  - Hibernate
  - HQL injection
  - JPQL injection
  - createQuery()
  - parameter binding
  - SQL injection
---

# HQL injection

Hibernate Query Language (HQL)—and the JPA equivalent JPQL—is an object-oriented query language that Hibernate translates to SQL. It supports named/positional **parameters**, but when application code concatenates user input into the query string passed to `createQuery()`, those protections are skipped and the query becomes injectable.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

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

HQL supports subqueries and `UNION`-like retrieval through entity navigation, so you can pivot to other mapped entities:

```
' OR username='admin' --
xyz' UNION SELECT u.password FROM User u WHERE '1'='1
```

Because HQL resolves field access against the mapping, you can read sensitive properties of related entities the query never intended to expose (e.g. `user.credentials.passwordHash` via association paths). HQL lacks some raw-SQL constructs, so where HQL is limited, pivot to the native-query sink (see [Native SQL Injection](native-sql-injection.md)) if the application also exposes one.

Blind extraction uses the same boolean/substring inference as SQL injection—craft conditions on `SUBSTRING(u.password,1,1)='a'` and observe result differences. Hibernate underneath runs on any JDBC backend (PostgreSQL, MySQL, Oracle, SQL Server), so the generated SQL dialect follows the configured database.

## References

- [Hibernate ORM: HQL/JPQL](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#hql)
- [OWASP: Testing for ORM Injection](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
