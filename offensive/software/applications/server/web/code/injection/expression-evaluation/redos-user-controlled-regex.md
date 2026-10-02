---
title: "ReDoS: user-controlled regular expressions and catastrophic backtracking"
description: Application endpoints that compile or execute regex patterns from HTTP input, enabling CPU exhaustion.
keywords:
  - ReDoS
  - regular expression denial of service
---

# ReDoS

## Context

**Regex** engines backtrack on ambiguous patterns. A short **evil** pattern + long **input** can burn CPU per request when the pattern is **user-supplied** or built from user fragments.

## Theory

Use **linear-time** matchers where possible, **timeout** regex execution, or **allowlist** pattern shapes.

## Practice

- Fuzz report or search endpoints that accept `regex=` parameters in a staging environment with CPU monitoring.
