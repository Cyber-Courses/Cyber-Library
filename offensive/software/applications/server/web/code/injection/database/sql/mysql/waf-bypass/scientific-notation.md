---
title: "MySQL WAF bypass: scientific notation and numeric obfuscation (1e1, 8e0)"
description: Representing numbers without digit literals where filters block specific characters.
keywords:
  - WAF bypass
  - MySQL SQL injection
---

# Scientific notation

## Context

Library Structure **Scientific Notation** tricks for numeric contexts when digit filters apply to strings but not numeric literals.

## See also

- [WAF bypass (parent)](index.md)
