---
title: "Sequelize injection"
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
