---
title: "ORM injection"
description: "Object-relational mappers parameterize normal queries, but every one exposes escape hatches that reopen injection — organized by framework."
keywords:
  - ORM injection
  - object-relational mapping
  - raw SQL
  - query builder
  - second-order injection
---

# ORM

Object-relational mappers parameterize ordinary queries, but every ORM exposes **escape hatches** — raw-SQL methods, string-built query fragments, and user-controlled identifiers or operators — that reintroduce injection. Because the vulnerable APIs differ per framework, this subtree is organized by framework: the sink in Django is not the sink in Hibernate or Sequelize.
