---
title: "MySQL WAF bypass: MySQL versioned comments (/*!50000 … */)"
description: Conditional comment execution hiding keywords from naive WAF parsers.
keywords:
  - WAF bypass
  - MySQL SQL injection
---

# Conditional comments

## Context

Library Structure **Conditional Comments**: `/*!50000SELECT*/` style tokens executed only on compatible servers, use in **authorized** fuzzing only.
