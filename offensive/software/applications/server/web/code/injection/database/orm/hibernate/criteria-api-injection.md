---
title: "Hibernate Criteria API injection: sqlRestriction and dynamic property names"
description: The Criteria API is parameter-safe for values, but Restrictions.sqlRestriction() embeds raw SQL and user-controlled property names reach unintended columns.
keywords:
  - Hibernate
  - Criteria API
  - sqlRestriction
  - Restrictions
  - property injection
  - SQL injection
---

# Criteria API injection

The Hibernate Criteria API builds queries programmatically and binds **values** as parameters, so the usual `Restrictions.eq("name", userValue)` call is not value-injectable. Two other avenues make it a sink: the raw-SQL escape hatch `Restrictions.sqlRestriction()`, and user-controlled **property or alias names** passed into criteria construction.

## Vulnerable patterns

**Raw SQL via `sqlRestriction`**, the fragment is spliced into the generated SQL:

```java
criteria.add(Restrictions.sqlRestriction("name = '" + name + "'"));
```

**User-controlled property/column name**, not parameterizable, so often concatenated:

```java
String sortCol = request.getParameter("sort");
criteria.addOrder(Order.asc(sortCol));              // column context
criteria.add(Restrictions.eq(userField, value));    // field chosen by user
```

## Exploitation

`sqlRestriction` is raw SQL, so inject as into a `WHERE` fragment:

```
x' OR '1'='1
x') UNION SELECT password FROM users --
x' OR (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a
```

`sqlRestriction` supports `?` placeholders for values; when they are skipped in favor of concatenation, the fragment is fully attacker-controlled.

For **property/column** contexts (`addOrder`, dynamic `eq`), there is no string to break out of, abuse it as a column-name injection: steer sorting to reveal ordering oracles, or reach association paths/columns the query never meant to expose. Modern code using the JPA **Criteria** (`CriteriaBuilder`) is safer for values but can still concatenate column names or fall back to `sqlRestriction`-style raw fragments.

## References

- [Hibernate ORM: Legacy Criteria / Restrictions](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
