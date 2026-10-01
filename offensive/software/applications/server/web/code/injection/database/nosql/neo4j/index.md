---
title: "Neo4j Cypher injection"
description: "String-built Cypher against Neo4j graph databases lets an attacker alter MATCH patterns, cross label boundaries, abuse APOC, and infer data blindly."
keywords:
  - Neo4j
  - Cypher injection
  - graph database injection
  - APOC abuse
  - blind graph injection
---

# Neo4j

Neo4j is a graph database queried with **Cypher**, and Cypher is injectable for the same reason SQL is: when application code concatenates user input into a query instead of binding parameters (`$param`), the input is parsed as query syntax. An attacker who reaches a string-built `MATCH` or `WHERE` can rewrite patterns and predicates, break out of a string literal, and append further clauses with `WITH`, `MATCH`, `CALL`, and comments (`//`, `/* */`).

Because Cypher traverses a single property graph with no table boundaries, injection crosses **labels** freely: one injected `MATCH (n) RETURN n` can enumerate every node, label, property, and relationship. Where the `apoc` procedure library is installed, `CALL` injection extends reach to SSRF, outbound exfiltration, and command execution. When no rows are reflected, boolean and time-based inference recover data blindly.

> **Scope.** For authorized penetration tests, CTF labs, and assessment of systems you own or are contracted to test.

This subtree covers the core injection mechanics, cross-label extraction, APOC procedure abuse, and blind inference.
