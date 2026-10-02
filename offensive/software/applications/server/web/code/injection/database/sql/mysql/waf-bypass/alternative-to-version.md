---
title: "MySQL WAF bypass: alternatives to version() and VERSION() (@@innodb_version, globals)"
description: Version fingerprinting without blocked function names—authorized WAF testing in staging.
keywords:
  - WAF bypass
  - MySQL SQL injection
---

# Alternative to VERSION()

## Context

Mirrors Library Structure **Alternative to VERSION**: `@@version`, `@@innodb_version`, `@@GLOBAL.version` and similar when `version()` is filtered.

## See also

- [WAF bypass (parent)](index.md)
