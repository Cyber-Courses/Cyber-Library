---
title: "ORM injection"
description: "Object-relational mappers parameterize normal queries, but every one exposes escape hatches that reopen injection, organized by framework."
keywords:
  - ORM injection
  - object-relational mapping
  - raw SQL
  - query builder
  - second-order injection
---

# ORM

Object-relational mappers parameterize ordinary queries, but every ORM exposes **escape hatches**, raw-SQL methods, string-built query fragments, and user-controlled identifiers or operators, that reintroduce injection. The vulnerable APIs are specific to each ORM, and ORMs are specific to a language, so this subtree is organized **by language, then by ORM**: the sink in Django is not the sink in Hibernate or Sequelize, and a new ORM slots under the language it belongs to rather than into a flat list.

## Organized by language

- **[JavaScript](javascript/index.md)**: Sequelize, Prisma, Drizzle, and other Node and TypeScript query builders, where the raw-query escape hatch and attacker-controlled operator or filter objects are the sinks.
- **[Python](python/index.md)**: Django ORM and SQLAlchemy, where `raw`/`extra`, `text()`, and lookup or filter construction from input bypass binding.
- **[Java](java/index.md)**: Hibernate and JPA, where HQL/JPQL string building, native queries, and dynamic criteria defeat parameters.
- **[C#](c-sharp/index.md)**: Entity Framework, where raw-SQL APIs and unsafe string interpolation reach the database unparameterized.

Across all of them the pattern is the same: the ORM is safe on its typed, parameterized path, and injection lives in the API a developer reaches for when that path is inconvenient.
