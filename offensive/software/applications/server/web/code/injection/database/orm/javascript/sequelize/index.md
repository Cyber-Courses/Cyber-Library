---
title: "Sequelize injection"
order: 1
description: "Injection through concatenated sequelize.query(), identifier interpolation, and attacker-controlled operator objects."
keywords:
  - Sequelize
  - Node.js ORM
  - raw query
  - operator injection
  - replacements
---

# Sequelize

Sequelize offers `replacements` and `bind` for safe parameterization, but concatenated `sequelize.query()` strings and interpolated identifiers such as a dynamic `ORDER BY` reintroduce injection in Node.js applications. A related case, attacker-controlled **operator objects** in a `where` clause, applies only where legacy string operator aliases are enabled or the app maps input to Sequelize's `Op` symbols; current defaults do not interpret JSON keys like `$ne` as operators.

## Pages

- **[Operator injection](operator-injection.md)**: When request JSON is passed into a Sequelize where clause, attacker-controlled operator keys ($gt, $ne, $like) alter query logic, authentication bypass and d...
- **[Raw query injection](raw-query-injection.md)**: sequelize.query() runs raw SQL against the backend; concatenating request data into it instead of using replacements or bind yields SQL injection in Node.js...
- **[Replacement injection](replacement-injection.md)**: Mixing :replacements with string concatenation, or feeding attacker-controlled identifiers through replacements, reintroduces SQL injection in Sequelize raw...

## Tools

- **sqlmap**: exploiting concatenated sequelize.query() raw-SQL sinks.
- **Burp Repeater**: testing interpolated identifiers and operator objects in where clauses.

## References

- Sequelize documentation: Raw queries
- OWASP: SQL Injection
