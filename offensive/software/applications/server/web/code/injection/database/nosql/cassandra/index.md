---
title: "Cassandra injection"
description: "String-built CQL against Apache Cassandra lets an attacker break out of quoted values, add predicates, force cluster-wide scans, and smuggle extra writes into a wide-column store."
keywords:
  - Cassandra injection
  - CQL injection
  - wide-column store
  - ALLOW FILTERING
  - batch statement injection
---

# Cassandra

Apache Cassandra is a distributed wide-column store queried with **CQL**, a SQL-like language. CQL is injectable for the same reason SQL is: when application code concatenates user input into a statement instead of binding parameters (`?` or `:name`), the input is parsed as query syntax rather than treated as a value. An attacker who reaches a string-built `WHERE` can close a quoted value, relax predicates, append `ALLOW FILTERING` to escape the partition the query intended, and in some drivers smuggle extra writes through a `BEGIN BATCH`.

CQL is deliberately narrow. It has no `UNION`, no subqueries, no `OR` across arbitrary columns, and no sleep or benchmark function, so the relational tricks that drive classic SQLi do not apply here. The real primitives are quoted-value breakout, extra predicates, and `ALLOW FILTERING` to turn a partition read into a full-cluster scan. Where the data store runs user-defined functions, injected `CREATE FUNCTION` can reach code execution inside the Cassandra JVM. This subtree covers the CQL mechanics, batch and UDF abuse, and blind and error-based inference.
