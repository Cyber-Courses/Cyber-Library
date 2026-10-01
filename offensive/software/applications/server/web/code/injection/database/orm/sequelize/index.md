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

Sequelize offers `replacements` and `bind` for safe parameterization, but concatenated `sequelize.query()` strings, interpolated identifiers such as a dynamic `ORDER BY`, and attacker-controlled **operator objects** in a `where` clause all reintroduce injection in Node.js applications.
